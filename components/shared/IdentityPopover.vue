<template>
  <v-tippy class="tippy-container" interactive :animate-fill="false" placement="bottom" :delay="[100, 800]">
    <template v-slot:trigger>
      <slot name="trigger" />
    </template>
    <div class="popover-content-container p-4">
      <div class="is-flex mb-3">
        <div class="is-align-self-flex-start">
          <!-- <Identicon
            :size="60"
            :theme="'polkadot'"
            :value="identity.address"
            class="popover-image avatar mr-5"
          /> -->
        </div>
        <div>
          <p class="has-text-weight-bold is-size-4 mb-1">
            {{ identity.display }}
          </p>
          <p class="is-size-7">
            {{ shortenedAddress }}
            <b-icon
              icon="copy"
              size="is-small"
              class="copy-icon"
              v-clipboard:copy="identity.address"
              @click.native="toast('Copied to clipboard')"
            ></b-icon>
          </p>
          <div class="py-1 is-size-7 py-3">
            Lorem ipsum dolor sit, amet consectetur adipisicing elit. Ratione soluta obcaecati
          </div>
          <p class="is-size-7 is-flex is-align-items-center pb-3">
            <b-icon
              icon="clock"
              size="is-small"
            />
            <span class="ml-2">Started minting {{ formattedTimeToNow }}</span>
          </p>
        </div>
      </div>

      <hr style="height: 1px;" class="m-0">

      <div style="" class="popover-stats-container pt-3">
        <div class="has-text-centered">
          <p class="has-text-weight-bold is-size-6">{{ totalCollected }}</p>
          <span class="is-size-7">Bought</span>
        </div>
        <div class="has-text-centered">
          <p class="has-text-weight-bold is-size-6">{{ totalCreated }}</p>
          <span class="is-size-7">Created</span>
        </div>
        <div class="has-text-centered">
          <p class="has-text-weight-bold is-size-6">{{ totalSold }}</p>
          <span class="is-size-7">Sold</span>
        </div>
      </div>
    </div>
  </v-tippy>
</template>

<script lang="ts" >
import { Component, mixins, Prop, Vue } from 'nuxt-property-decorator'
import {formatDistanceToNow} from 'date-fns'
import { GenericAccountId } from '@polkadot/types/generic/AccountId'
import { notificationTypes, showNotification } from '@/utils/notification'
import nftStatsByIssuer from '@/queries/nftStatsByIssuer.graphql'
import shortAddress from '@/utils/shortAddress'
import Identicon from '@polkadot/vue-identicon'
import PrefixMixin from '~/utils/mixins/prefixMixin'

type Address = string | GenericAccountId | undefined;
type IdentityFields = Record<string, string>;

@Component({
  components: {
    Identicon
  }
})
export default class IdentityPopover extends mixins(PrefixMixin) {
  @Prop() public identity!: IdentityFields

  protected totalCreated = 0
  protected totalCollected = 0
  protected totalSold = 0
  protected firstMintDate = new Date()

  get shortenedAddress(): Address {
    return shortAddress(this.resolveAddress(this.identity.address))
  }

  private resolveAddress(account: Address): string {
    return account instanceof GenericAccountId ? account.toString() : account || ''
  }

  public toast(message: string): void {
    this.$buefy.toast.open(message)
  }

  get formattedTimeToNow() {
    return this.firstMintDate ? formatDistanceToNow(new Date(this.firstMintDate), { addSuffix: true }) : ''
  }

  public async mounted() {
    await this.fetchNFTStats()
  }

  protected async fetchNFTStats() {
    try {
      const query = this.urlPrefix === 'rmrk' ? await import('@/queries/nftStatsByIssuer.graphql') : await import('@/queries/unique/nftStatsByIssuer.graphql')
      this.$apollo.addSmartQuery('collections', {
        query: query.default,
        manual: true,
        client: this.urlPrefix,
        loadingKey: 'isLoading',
        result: this.handleResult,
        variables: () => {
          return {
            account: this.identity.address,
          }
        },
        fetchPolicy: 'cache-and-network'
      })
    } catch (e) {
      showNotification(`${e}`, notificationTypes.danger)
      console.warn(e)
    }
  }

  protected async handleResult({ data }: any) {
    if (data) {
      this.totalCreated = data.nFTCreated.totalCount
      this.totalCollected = data.nFTCollected.totalCount
      this.totalSold = data.nFTSold.totalCount

      if (data?.firstMint?.nodes.length > 0) {
        this.firstMintDate = data.firstMint.nodes[0].collection.createdAt
      }
    }
  }
}
</script>

<style scoped>
.tippy-container {
  display: inline-block
}

.popover-content-container {
  border: 1px solid white;
  max-width: 350px
}

.popover-image {
  min-width: 60px;
}

.popover-stats-container {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

.copy-icon {
  cursor: pointer;
}
</style>
