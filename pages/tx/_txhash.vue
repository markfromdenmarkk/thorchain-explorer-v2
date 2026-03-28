<template>
  <Page>
    <div class="tx-header">
      <div class="item tx-id">
        <span class="mono">
          {{ $route.params.txhash }}
        </span>
      </div>
      <div class="item">
        <Copy :str-copy="$route.params.txhash" />
      </div>
      <div id="qrcode" class="item">
        <qr-btn :qrcode="$route.params.txhash"></qr-btn>
      </div>
    </div>
    <div
      v-for="(overview, index) in contractOverviews"
      :key="'contract-overview-' + index"
      class="contract-overview card-bg"
    >
      <div class="contract-overview__header">
        <div>
          <small class="contract-overview__eyebrow">Contract transaction</small>
          <h2 class="contract-overview__title">{{ overview.title }}</h2>
        </div>
        <div
          :class="[
            'contract-overview__status',
            overview.status === 'Success'
              ? 'contract-overview__status--success'
              : 'contract-overview__status--error',
          ]"
        >
          {{ overview.status }}
        </div>
      </div>

      <div
        v-if="overview.stats && overview.stats.length > 0"
        class="contract-overview__stats"
      >
        <div
          v-for="(stat, statIndex) in overview.stats"
          :key="'contract-stat-' + index + '-' + statIndex"
          class="contract-overview__stat"
        >
          <small class="contract-overview__stat-label">{{ stat.label }}</small>
          <div
            :class="[
              'contract-overview__stat-value',
              {
                'contract-overview__stat-value--muted': stat.isUnavailable,
                'contract-overview__stat-value--compact': true,
              },
            ]"
          >
            <template v-if="stat.party">
              <nuxt-link
                :to="`/address/${stat.party.address}`"
                class="contract-overview__party-link"
              >
                <template v-if="stat.party.name">
                  {{ stat.party.name }}
                </template>
                <template v-else>
                  {{ stat.party.shortAddress }}
                </template>
              </nuxt-link>
            </template>
            <template v-else>
              <span>{{ stat.value }}</span>
              <AssetIcon
                v-if="stat.asset"
                class="contract-overview__stat-icon"
                :asset="stat.asset"
                :height="'1.15rem'"
                :chain="false"
              />
            </template>
          </div>
          <small
            v-if="stat.subtext || stat.subtextParty"
            class="contract-overview__stat-subtext"
          >
            <template v-if="stat.subtextParty">
              <nuxt-link
                :to="`/address/${stat.subtextParty.address}`"
                class="contract-overview__party-link"
              >
                <template v-if="stat.subtextParty.name">
                  {{ stat.subtextParty.name }}
                </template>
                <template v-else>
                  {{ stat.subtextParty.shortAddress }}
                </template>
              </nuxt-link>
            </template>
            <template v-else>
              {{ stat.subtext }}
            </template>
          </small>
        </div>
      </div>

      <div class="contract-overview__content">
        <div
          v-if="overview.assetFlow && overview.assetFlow.length > 0"
          class="contract-overview__panel"
        >
          <button
            type="button"
            class="contract-overview__accordion-toggle"
            @click="toggleContractOverviewSection(index, 'asset-flow')"
          >
            <strong class="contract-overview__accordion-left">
              Asset Flow
              <angle-icon
              :class="[
                'contract-overview__accordion-icon',
                {
                  'contract-overview__accordion-icon--open':
                    isContractOverviewSectionOpen(index, 'asset-flow'),
                },
              ]"
              />
            </strong>
          </button>
          <div
            v-show="isContractOverviewSectionOpen(index, 'asset-flow')"
            :class="[
              'contract-overview__accordion-body',
              'contract-overview__flow-list',
              {
                'contract-overview__accordion-body--open':
                  isContractOverviewSectionOpen(index, 'asset-flow'),
              },
            ]"
          >
            <div
              v-for="(flow, flowIndex) in overview.assetFlow"
              :key="'contract-flow-' + index + '-' + flowIndex"
              class="contract-overview__flow-item"
            >
              <div class="contract-overview__flow-party">
                <small>From</small>
                <div>
                  <nuxt-link
                    v-if="flow.from.address"
                    :to="`/address/${flow.from.address}`"
                    class="contract-overview__party-link"
                  >
                    <template v-if="flow.from.name">
                      {{ flow.from.name }}
                    </template>
                    <template v-else>
                      {{ flow.from.shortAddress }}
                    </template>
                  </nuxt-link>
                  <template v-else>
                    {{ flow.from.text }}
                  </template>
                </div>
              </div>
              <div class="contract-overview__flow-center">
                <div class="contract-overview__flow-amount">
                  {{ flow.amount }}
                </div>
                <div class="contract-overview__flow-token">
                  <AssetIcon
                    v-if="flow.asset"
                    :asset="flow.asset"
                    :height="'1.15rem'"
                    :chain="false"
                  />
                  <small>{{ flow.token }}</small>
                </div>
              </div>
              <div class="contract-overview__flow-party">
                <small>To</small>
                <div>
                  <nuxt-link
                    v-if="flow.to.address"
                    :to="`/address/${flow.to.address}`"
                    class="contract-overview__party-link"
                  >
                    <template v-if="flow.to.name">
                      {{ flow.to.name }}
                    </template>
                    <template v-else>
                      {{ flow.to.shortAddress }}
                    </template>
                  </nuxt-link>
                  <template v-else>
                    {{ flow.to.text }}
                  </template>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div
          v-if="overview.execution && overview.execution.length > 0"
          class="contract-overview__panel"
        >
          <button
            type="button"
            class="contract-overview__accordion-toggle"
            @click="toggleContractOverviewSection(index, 'contracts-addresses')"
          >
            <strong class="contract-overview__accordion-left">
              Contracts and Addresses
              <angle-icon
              :class="[
                'contract-overview__accordion-icon',
                {
                  'contract-overview__accordion-icon--open':
                    isContractOverviewSectionOpen(index, 'contracts-addresses'),
                },
              ]"
              />
            </strong>
          </button>
          <div
            v-show="isContractOverviewSectionOpen(index, 'contracts-addresses')"
            :class="[
              'contract-overview__accordion-body',
              'contract-overview__detail-list',
              {
                'contract-overview__accordion-body--open':
                  isContractOverviewSectionOpen(index, 'contracts-addresses'),
              },
            ]"
          >
            <div
              v-for="(detail, detailIndex) in overview.execution"
              :key="'contract-detail-' + index + '-' + detailIndex"
              :class="[
                'contract-overview__detail-item',
                {
                  'contract-overview__detail-item--full': detail.fullWidth,
                  'contract-overview__detail-item--multiline': detail.multiline,
                },
              ]"
            >
              <small>{{ detail.label }}</small>
              <div>
                <template v-if="detail.parties && detail.parties.length > 0">
                  <div
                    v-for="(party, partyIndex) in detail.parties"
                    :key="'contract-detail-party-' + index + '-' + detailIndex + '-' + partyIndex"
                  >
                    <nuxt-link
                      v-if="party.address"
                      :to="`/address/${party.address}`"
                      class="contract-overview__party-link"
                    >
                      <template v-if="party.name">
                        {{ party.name }}
                      </template>
                      <template v-else>
                        {{ party.shortAddress }}
                      </template>
                    </nuxt-link>
                    <template v-else>
                      {{ party.text }}
                    </template>
                  </div>
                </template>
                <template v-else-if="detail.party">
                  <nuxt-link
                    v-if="detail.party.address"
                    :to="`/address/${detail.party.address}`"
                    class="contract-overview__party-link"
                  >
                    <template v-if="detail.party.name">
                      {{ detail.party.name }}
                    </template>
                    <template v-else>
                      {{ detail.party.shortAddress }}
                    </template>
                  </nuxt-link>
                  <template v-else>
                    {{ detail.party.text }}
                  </template>
                </template>
                <template v-else>
                  {{ detail.value }}
                </template>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <template v-if="!isError && !isLoading && pools">
      <template v-if="cards && cards.length > 0">
        <tx-card v-for="(c, i) in cards" :key="i" :tx-data="c.details">
          <template
            v-for="(s, j) in c.accordions.filter(
              (c) => c.data.title && !c.data.hide
            )"
            #[s.name]
          >
            <accordion
              :key="i + '.' + j"
              :title="s.data.title"
              :pending="s.data.pending"
              :done="s.data.done"
              :remaining-time="s.data.remainingTime"
              :total-time="s.data.totalTime"
              :asset="s.data.asset"
              :stacks="s.data.stacks"
              :error="s.data.error"
              :show-at-first="true"
              :attributes="s.data.attributes"
              :events="s.data.events"
            />
          </template>
        </tx-card>
        <streaming-swap
          v-if="inboundHash"
          :inbound-hash="inboundHash"
          :quote="quote"
          :height="height"
        />
      </template>
      <tx-loader v-else></tx-loader>
    </template>
    <div v-else-if="isError" class="notify-card card-bg">
      <h3>{{ error.title }}</h3>
      <span>{{ error.message }}</span>
      <DisconnectIcon class="disconnect-icon" />
    </div>
    <div v-else-if="isLoading && !isError">
      <tx-loader />
    </div>
  </Page>
</template>

<script>
import moment from 'moment'
import { orderBy, groupBy, sumBy } from 'lodash'
import { mapGetters } from 'vuex'
import rujiraMainnetContracts from '@/data/contracts/rujira-mainnet.json'
import streamingSwap from './components/streamingSwap.vue'
import txCard from './components/txCard.vue'
import { createCard as buildCard } from './state/cardBuilder.js'
import {
  BUILDERS as BUILDERS_MODULE,
  createFailedState as createFailedStateBuilder,
} from './state/builders.js'
import AssetIcon from '~/components/AssetIcon.vue'
import DisconnectIcon from '~/assets/images/disconnect.svg?inline'
import AngleIcon from '~/assets/images/angle-down.svg?inline'
import {
  assetFromString,
  assetToTrade,
  assetToSecure,
  tradeToAsset,
  assetToString,
  securedToAsset,
  sumAffiliateFee,
} from '~/utils'
import Accordion from '~/components/Accordion.vue'

export default {
  components: {
    DisconnectIcon,
    AssetIcon,
    AngleIcon,
    streamingSwap,
    txCard,
    Accordion,
  },
  data() {
    return {
      tx: undefined,
      extraSwapDetails: undefined,
      isLoading: true,
      isError: false,
      copyText: 'Copy',
      loadingPercentage: 0,
      progressText: '',
      txFormatted: undefined,
      error: {
        title: "Couldn't find the Transaction",
        message: 'Something bad happened.',
      },
      updateInterval: undefined,
      cards: [],
      inboundHash: undefined,
      thorStatus: undefined,
      thorHeight: 0,
      quote: undefined,
      height: undefined,
      contractOverviewSections: {},
    }
  },
  head: {
    title: 'THORChain Network Explorer | TX',
  },
  computed: {
    ...mapGetters({
      chainsHeight: 'getChainsHeight',
      pools: 'getPools',
      runePrice: 'getRunePrice',
    }),
    contractOverviews() {
      return (this.cards ?? [])
        .map((card) => card?.details?.overview)
        .filter(Boolean)
    },
    rujiraRegistry() {
      return process.env.NETWORK === 'mainnet' ? rujiraMainnetContracts : null
    },
  },
  async mounted() {
    let txHash = this.$route.params.txhash
    if (txHash.toLowerCase().startsWith('0x')) {
      txHash = txHash.slice(2)
    }

    let isPending = false
    try {
      isPending = await this.fetchTx(txHash)
    } catch (error) {
      if (txHash.length <= 45) {
        const addrTxs = await this.$api.getAddress(txHash, 0)
        if (addrTxs?.data?.actions?.length > 0) {
          this.gotoAddr(this.$route.params.txhash)
        }
      }
      console.error(error)
      this.isError = true
      this.isLoading = false
      return
    }

    // if has no outbound
    if (isPending) {
      const uI = setInterval(async () => {
        try {
          isPending = await this.fetchTx(txHash)
        } catch (error) {
          if (txHash.length <= 45) {
            const addrTxs = await this.$api.getAddress(txHash, 0)
            if (addrTxs?.data?.actions?.length > 0) {
              this.gotoAddr(this.$route.params.txhash)
            }
          }
          console.error(error)
          this.isError = true
          this.isLoading = false
          return
        }
        if (!isPending) {
          clearInterval(uI)
        }
      }, 5000)

      this.updateInterval = uI
    }
  },
  destroyed() {
    this.clearIntervalId(this.updateInterval)
  },
  methods: {
    // TODO: check hash in saver with streaming
    async fetchTx(hash) {
      if (!this.pools) {
        return true
      }

      // Here the hash can be outbound but the inbound should be caught if it's not
      // Get Midgard details
      const md = (
        await this.$api.getActions({ txid: hash }).catch((e) => {
          if (e?.response?.status === 404) {
            this.error.message =
              'Transaction is not found in Midgard. Please make sure the correct transaction hash or account address is inserted.'
          }
        })
      )?.data

      // See if the hash is outbound
      const swapAction = md?.actions?.find((a) => a.type === 'swap')
      if (swapAction) {
        hash = swapAction.in?.[0]?.txID
      }

      // get inbound hash
      this.inboundHash = hash

      // Get THORNode details
      let archival = false
      let thorRes = await this.$api.getThornodeDetailTx(hash).catch((e) => {
        if (e?.response?.status / 200 !== 1) {
          this.error.message =
            'Transaction is not found in Thornode. Please make sure the correct transaction hash or account address is inserted.'
        }
      })

      if (!thorRes) {
        thorRes = await this.$api.getThornodeArchiveTx(hash).catch((e) => {
          if (e?.response?.status / 200 !== 1) {
            this.error.message =
              'Transaction is not found in Thornode. Please make sure the correct transaction hash or account address is inserted.'
          }
        })
        archival = true
      }

      // Assign header and data if available
      let td, tdh
      if (thorRes) {
        td = thorRes.data
        tdh = thorRes.headers
      }

      let ts = (
        await this.$api.getTxStatus(hash).catch((e) => {
          if (e?.response?.status / 200 !== 1) {
            this.error.message =
              "Can't find transaction status. Please make sure the correct transaction hash or account address is inserted."
          }
        })
      )?.data

      if (archival) {
        ts = (
          await this.$api.getTxArchiveStatus(hash).catch((e) => {
            if (e?.response?.status / 200 !== 1) {
              this.error.message =
                "Can't find transaction status. Please make sure the correct transaction hash or account address is inserted."
            }
          })
        )?.data
      }

      this.thorStatus = ts
      this.isLoading = false

      const nt = md?.actions?.find((a) => a.type === 'send')
      const memo = this.parseMemo(td?.tx?.tx?.memo)
      // TODO: add proper error handling
      if (nt && (!memo.type || memo.type === 'unknown')) {
        this.createNativeTx(nt)
        return false
      } else {
        if (tdh) {
          this.thorHeight = parseInt(tdh['x-thorchain-height'] ?? 0)
        }
        this.createTxState(md, td, ts, tdh, this.pools)
        return this.isTxInPending(ts, md)
      }
    },
    isTxInPending(thorStatus, actions) {
      const memo = this.parseMemo(thorStatus?.tx?.memo)

      const userAddresses = new Set([
        thorStatus?.tx?.from_address?.toLowerCase(),
        memo.destAddr?.toLowerCase(),
      ])

      let outTxs = thorStatus?.out_txs?.filter((tx) =>
        userAddresses.has(tx?.to_address?.toLowerCase())
      )

      if (!outTxs) {
        outTxs = thorStatus?.planned_out_txs
          ?.filter((tx) => userAddresses.has(tx.to_address.toLowerCase()))
          .map((tx) => ({
            ...tx,
            coins: [{ amount: tx.coin.amount, asset: tx.coin.asset }],
          }))
      }

      const outAsset = this.parseMemoAsset(
        outTxs?.length > 0 ? outTxs[0]?.coins?.[0]?.asset : memo?.asset,
        this.pools
      )

      if (actions && actions.actions?.length > 0) {
        if (memo.type !== 'swap') {
          const success = actions.actions.some((e, i) => e.status === 'success')
          if (success) {
            return false
          }
        }
        let ta
        if (outAsset) {
          ta = assetFromString(outAsset)
        }
        const isRefund = actions.actions.some((e, i) => e.type === 'refund')
        if (isRefund && (ta?.synth || ta?.trade || ta?.secure)) {
          return false
        }
      }

      const inboundFinalised = thorStatus?.stages?.inbound_finalised?.completed
      let actionFinalised = true
      if (memo.type === 'swap') {
        actionFinalised =
          (outAsset?.chain !== 'THOR' &&
            thorStatus?.stages.swap_finalised?.completed) ||
          !thorStatus?.stages.swap_status?.pending
      }
      const outboundFinalised =
        (thorStatus?.stages.outbound_signed?.completed ||
          outAsset?.chain === 'THOR' ||
          outAsset?.synth ||
          outAsset?.trade ||
          outAsset?.secure) &&
        (thorStatus?.stages?.outbound_delay?.completed ?? true)

      return !inboundFinalised || !actionFinalised || !outboundFinalised
    },
    getBuilderContext() {
      return {
        parseMemo: this.parseMemo.bind(this),
        parseMemoAsset: this.parseMemoAsset.bind(this),
        pools: this.pools,
      }
    },
    getCardContext() {
      return {
        amountToUSD: this.amountToUSD.bind(this),
        formatAddress: this.formatAddress.bind(this),
        showAsset: this.showAsset.bind(this),
        formatCurrency: this.formatCurrency.bind(this),
        formatSmallCurrency: this.formatSmallCurrency.bind(this),
        percentageFormat: this.percentageFormat.bind(this),
        baseAmountFormatOrZero: this.baseAmountFormatOrZero.bind(this),
        normalFormat: this.normalFormat.bind(this),
        getInboundStages: this.getInboundStages.bind(this),
        getOutboundStages: this.getOutboundStages.bind(this),
        chainsHeight: this.chainsHeight,
        blockSeconds: this.blockSeconds.bind(this),
        height: this.height,
        runePrice: this.runePrice,
        pools: this.pools,
        pluralize: (value, singular, options) =>
          this.$options.filters.pluralize(value, singular, options),
      }
    },
    createCard(cardBase, accordions) {
      return buildCard(cardBase, accordions, this.getCardContext())
    },
    async createTxState(midgardAction, thorTx, thorStatus, thorHeader, pools) {
      const memo = this.parseMemo(thorTx?.tx?.tx?.memo)

      if (memo.type === 'outbound') {
        this.gotoTx(memo.hash)
        return
      }

      // Swap: fetch quote when pending, then build state
      if (memo.type === 'swap') {
        const inAsset = this.parseMemoAsset(
          thorStatus?.tx?.coins?.[0]?.asset,
          this.pools
        )
        const inAmount = parseInt(thorStatus?.tx?.coins?.[0]?.amount ?? 0)
        const outAsset = this.parseMemoAsset(memo?.asset, this.pools)
        const affiliateFee = sumAffiliateFee(memo.fee || 0)
        if (thorStatus?.stages.swap_status?.pending && !this.quote) {
          try {
            const { data: quoteData } = await this.$api.getQuote({
              amount: inAmount,
              from_asset: inAsset ? assetToString(inAsset) : '',
              to_asset: outAsset ? assetToString(outAsset) : '',
              destination: memo.destAddr,
              streaming_interval:
                thorStatus?.stages.swap_status?.streaming?.interval ||
                memo.interval,
              ...(affiliateFee && { affiliate: memo.affiliate }),
              ...(affiliateFee && { affiliate_bps: affiliateFee }),
              height: midgardAction?.actions[0]?.height,
            })
            this.quote = quoteData
          } catch (error) {
            console.error(error)
          }
        }
        const { cards, accordions } = await this.createSwapState(
          thorStatus,
          thorTx,
          midgardAction,
          memo,
          thorHeader
        )
        this.$set(this, 'cards', [this.createCard(cards, accordions)])
        this.appendContractCards(midgardAction, thorStatus, thorTx, memo)
        return
      }

      // Add liquidity (possibly with asymmetry second card)
      if (memo.type === 'add') {
        const finalCards = []
        const { cards, accordions } = this.createAddLiquidityState(
          thorStatus,
          midgardAction,
          thorTx,
          memo
        )
        finalCards.push(this.createCard(cards, accordions))
        if (memo.asymmetry) {
          const ts = await this.getOtherActionHash(midgardAction, thorStatus)
          if (ts?.tx) {
            const m = this.parseMemo(ts.tx?.memo)
            const addState = this.createAddLiquidityState(
              ts,
              midgardAction,
              thorTx,
              m
            )
            finalCards.push(
              this.createCard(addState.cards, addState.accordions)
            )
          }
        }
        this.$set(this, 'cards', finalCards)
        this.appendContractCards(midgardAction, thorStatus, thorTx, memo)
        return
      }

      // Registry: memo.type -> builder (method name or function from module)
      const BUILDERS = {
        withdraw: 'createRemoveLiquidityState',
        runePoolWithdraw: 'createRunePoolWithdraw',
        runePoolDeposit: 'createRunePoolDeposit',
        tradeWithdraw: 'createTradeWithdrawState',
        secureWithdraw: 'createTradeWithdrawState',
        tradeDeposit: 'createTradeDepositState',
        secureDeposit: 'createTradeDepositState',
        bond: 'createBondState',
        unbound: 'createUnbondState',
        thorname: BUILDERS_MODULE.thorname,
        loanRepayment: 'createLoanRepayment',
        tcyUnstake: 'createTCYUnstake',
        tcyStake: 'createTCYStake',
      }

      const builder = BUILDERS[memo.type]
      if (builder) {
        const result =
          typeof builder === 'function'
            ? builder(
                thorStatus,
                midgardAction,
                thorTx,
                memo,
                this.getBuilderContext()
              )
            : this[builder](thorStatus, midgardAction, thorTx, memo)
        this.$set(this, 'cards', [
          this.createCard(result.cards, result.accordions),
        ])
        this.appendContractCards(midgardAction, thorStatus, thorTx, memo)
        return
      }

      // Failed deposit (by action type, not memo)
      if (
        midgardAction?.actions?.length > 0 &&
        midgardAction.actions[0]?.type === 'failed'
      ) {
        const { cards, accordions } = createFailedStateBuilder(
          thorStatus,
          midgardAction,
          thorTx,
          memo,
          this.getBuilderContext()
        )
        this.$set(this, 'cards', [this.createCard(cards, accordions)])
        this.appendContractCards(midgardAction, thorStatus, thorTx, memo)
        return
      }

      // Default: one card per action (createAbstractState). Skip contract
      // actions here; appendContractCards adds proper "Contract Event" cards.
      // Refund is no longer skipped so standalone refunds (e.g. empty memo) get a card.
      const finalCards = []
      for (let i = 0; i < midgardAction?.actions?.length; i++) {
        const action = midgardAction.actions[i]
        if (action?.type === 'contract') continue
        const { cards, accordions } = this.createAbstractState(
          thorStatus,
          action,
          thorTx,
          memo
        )
        finalCards.push(this.createCard(cards, accordions))
      }
      this.$set(this, 'cards', finalCards)
      this.appendContractCards(midgardAction, thorStatus, thorTx, memo)
    },
    appendContractCards(midgardAction, thorStatus, thorTx, memo) {
      if (!midgardAction?.actions?.map((a) => a.type).includes('contract')) {
        return
      }
      const contractCards = []
      for (let i = 0; i < midgardAction?.actions?.length; i++) {
        if (midgardAction.actions[i].type !== 'contract') continue
        const { cards, accordions } = this.createContractState(
          thorStatus,
          midgardAction.actions[i],
          thorTx,
          memo
        )
        contractCards.push(this.createCard(cards, accordions))
      }
      this.$set(this, 'cards', [...contractCards, ...this.cards])
    },
    getContractOverviewSectionKey(index, section) {
      return `${index}:${section}`
    },
    isContractOverviewSectionOpen(index, section) {
      const key = this.getContractOverviewSectionKey(index, section)
      return this.contractOverviewSections[key] === true
    },
    toggleContractOverviewSection(index, section) {
      const nextValue = !this.isContractOverviewSectionOpen(index, section)
      ;['asset-flow', 'contracts-addresses'].forEach((linkedSection) => {
        const key = this.getContractOverviewSectionKey(index, linkedSection)
        this.$set(this.contractOverviewSections, key, nextValue)
      })
    },
    buildContractSummary(action) {
      const contract = action?.metadata?.contract
      if (!contract) {
        return ''
      }

      const events = contract?.contractEvents ?? []
      const caller = action?.in?.[0]?.address
      const contractAddress =
        action?.out?.[0]?.address || this.getContractAddress(contract)
      const method = this.getContractMethod(contract)
      const succeeded = (contract?.code ?? 0) <= 0
      const callerText = caller ? this.formatContractParty(caller) : 'The caller'
      const transfers = this.extractContractTransfers(events)
      const networkFee = this.extractContractNetworkFee(events)
      const tradeDetails = this.extractContractTrade(events, transfers)
      const specificEvent = this.getPrimaryContractEventType(events)
      const strategyPath = this.getContractEventAttribute(events, ['path'])

      const summaryParts = []
      let intro = `${callerText} ${
        succeeded ? 'executed' : 'attempted to execute'
      } ${method}`
      if (contractAddress) {
        intro += ` on ${this.formatContractParty(contractAddress)}`
      }
      intro += succeeded ? ' successfully.' : '.'
      summaryParts.push(intro)

      if (tradeDetails) {
        let tradeText = `Processed ${tradeDetails.offerText}`
        if (tradeDetails.bidText) {
          tradeText += ` for ${tradeDetails.bidText}`
        }
        if (tradeDetails.rate) {
          tradeText += ` at a rate of ${tradeDetails.rate}`
        }
        tradeText += '.'
        summaryParts.push(tradeText)
      }

      const transferText = this.describeContractTransfers(transfers)
      if (transferText) {
        summaryParts.push(`Token flow: ${transferText}.`)
      } else if (!tradeDetails) {
        let fallbackText = 'The emitted events did not include explicit asset amounts or transfer legs.'
        if (strategyPath) {
          fallbackText = `This call triggered strategy path ${strategyPath}. ${fallbackText}`
        }
        summaryParts.push(fallbackText)
      }

      if (networkFee) {
        let feeText = `Network fee: ${networkFee.amountText}`
        if (networkFee.payer) {
          feeText += ` paid by ${this.formatContractParty(networkFee.payer)}`
        }
        feeText += '.'
        summaryParts.push(feeText)
      }

      if (specificEvent) {
        summaryParts.push(
          `Main contract event: ${this.humanizeContractLabel(specificEvent)}.`
        )
      }

      const reason = this.describeContractFailureReason(contract?.logs)
      if (!succeeded && reason) {
        summaryParts.push(`Failure reason: ${reason}.`)
      }

      return summaryParts.join(' ')
    },
    buildContractOverview(action, thorStatus) {
      const contract = action?.metadata?.contract
      if (!contract) {
        return null
      }

      const events = contract?.contractEvents ?? []
      const caller = action?.in?.[0]?.address
      const contractAddress =
        action?.out?.[0]?.address || this.getContractAddress(contract)
      const timestamp = action?.date ? moment.unix(action.date / 1e9) : null
      const networkFee = this.extractContractNetworkFee(events)
      const transfers = this.extractContractTransfers(events)
      const tradeDetails = this.extractContractTrade(events, transfers)
      const specificEvent = this.getPrimaryContractEventType(events)
      const touchedContracts = this.extractTouchedContracts(events)
      const status = (contract?.code ?? 0) <= 0 ? 'Success' : 'Failed'
      const methodLabel = this.getContractMethodDisplay(contract, events)
      const totalAmount = this.buildContractTotalAmountStat(
        action,
        contract,
        tradeDetails
      )
      const fallbackNetworkFee = this.extractThorchainNetworkFee(thorStatus)
      const feeSource = networkFee?.amountDisplay ? networkFee : fallbackNetworkFee
      const userTrade = this.extractContractUserTrade(caller, transfers)
      const amountOutStat = this.buildContractAmountOutStat(
        tradeDetails,
        userTrade,
        transfers
      )
      const amountInStat = this.buildContractAmountInStat(
        totalAmount,
        tradeDetails,
        userTrade,
        transfers
      )
      const contractStat = this.buildContractTopContractStat(contractAddress)
      const timestampStat = timestamp?.isValid()
        ? {
            label: 'Timestamp',
            value: timestamp.fromNow(),
            subtext: timestamp.format('L LT'),
          }
        : {
            label: 'Timestamp',
            value: 'Not exposed',
            subtext: 'No timestamp available for this action',
            isUnavailable: true,
          }

      const stats = [
        amountInStat,
        amountOutStat,
        contractStat,
        timestampStat,
      ]

      const execution = this.layoutContractExecutionDetails([
        feeSource?.payer
          ? {
              label: 'Fee Payer',
              party: this.buildContractParty(feeSource.payer),
              value: this.formatContractParty(feeSource.payer),
            }
          : null,
        caller
          ? {
              label: 'Caller',
              party: this.buildContractParty(caller),
              value: this.formatContractParty(caller),
            }
          : null,
        touchedContracts.length > 0
          ? {
              label: 'Contracts Touched',
              parties: touchedContracts
                .slice(0, 4)
                .map((address) => this.buildContractParty(address)),
              value: touchedContracts
                .slice(0, 4)
                .map((address) => this.formatContractParty(address))
                .join('\n'),
              multiline: true,
              fullWidth: true,
            }
          : null,
      ])

      const prioritizedTransfers = this.prioritizeContractAssetFlow(
        transfers,
        amountOutStat,
        amountInStat
      )
      const assetFlow = prioritizedTransfers
        .slice(0, 4)
        .map((transfer) => ({
          from: transfer.sender
            ? this.buildContractParty(transfer.sender)
            : { text: 'Unknown', address: '', name: '', shortAddress: '' },
          to: transfer.recipient
            ? this.buildContractParty(transfer.recipient)
            : { text: 'Unknown', address: '', name: '', shortAddress: '' },
          amount: transfer.amountText,
          token: transfer.token,
          asset: this.resolveContractFlowAsset(transfer.token),
        }))

      return {
        title:
          this.getRujiraContractMeta(contractAddress)?.name || 'Contract Event',
        method: methodLabel,
        status,
        stats,
        assetFlow,
        execution,
      }
    },
    getContractMethod(contract) {
      const rawMethod =
        this.getContractMessageName(contract?.msg) ||
        contract?.attributes?.action ||
        this.getContractEventAttribute(contract?.contractEvents, [
          'action',
          'method',
          'execute_msg',
          'message_type',
        ]) ||
        contract?.contractType

      if (!rawMethod) {
        return 'a contract method'
      }

      const method = this.humanizeContractLabel(rawMethod)
      return method ? `the ${method} method` : 'a contract method'
    },
    buildContractTotalAmountStat(action, contract, tradeDetails) {
      const fundsText = this.describeContractFunds(
        contract?.funds,
        action?.in?.[0]?.coins
      )

      if (tradeDetails?.offerText) {
        return {
          label: 'Total Amount',
          value: tradeDetails.offerText,
          subtext: tradeDetails.bidText
            ? `Outputs ${tradeDetails.bidText}`
            : 'Derived from emitted trade events',
        }
      }

      if (fundsText) {
        return {
          label: 'Total Amount',
          value: fundsText,
          subtext: 'Funds attached to the contract call',
        }
      }

      return {
        label: 'Total Amount',
        value: 'Not exposed',
        subtext: 'No amount found in attached funds or emitted events',
        isUnavailable: true,
      }
    },
    buildContractNetworkFeeStat(fee) {
      if (fee?.amountDisplay) {
        return {
          label: 'Network Fee',
          value: fee.amountDisplay,
          subtextParty: fee.payer ? this.buildContractParty(fee.payer) : null,
        }
      }

      return {
        label: 'Network Fee',
        value: 'Not exposed',
        subtext: 'No fee found in tx metadata',
        isUnavailable: true,
      }
    },
    buildContractTopContractStat(address) {
      const party = this.buildContractParty(address)

      if (party?.address) {
        return {
          label: 'Contract',
          value: party.name || party.shortAddress,
          party,
        }
      }

      return {
        label: 'Contract',
        value: 'Not exposed',
        subtext: 'No contract address found for this action',
        isUnavailable: true,
      }
    },
    buildContractAmountOutStat(tradeDetails, userTrade, transfers = []) {
      const amountOutValue = userTrade?.amountOut?.text || tradeDetails?.bidText || ''
      const excludedTokens = [
        userTrade?.amountIn?.token,
        tradeDetails?.offerToken,
        this.extractContractAmountToken(
          userTrade?.amountIn?.text || tradeDetails?.offerText || ''
        ),
      ].filter(Boolean)
      const amountOutToken =
        userTrade?.amountOut?.token ||
        tradeDetails?.bidToken ||
        this.extractContractAmountToken(amountOutValue) ||
        this.inferContractTradeTokenFromTransfers(transfers, amountOutValue, {
          excludeTokens: excludedTokens,
        })
      const amountOut = this.formatContractDisplayAmount(
        amountOutValue,
        amountOutToken
      )

      if (amountOut) {
        return {
          label: 'Amount Out',
          value: amountOut,
          asset: amountOutToken
            ? this.resolveContractFlowAsset(amountOutToken)
            : '',
        }
      }

      return {
        label: 'Amount Out',
        value: 'Not exposed',
        subtext: 'No output amount found in emitted events',
        isUnavailable: true,
      }
    },
    buildContractAmountInStat(totalAmount, tradeDetails, userTrade, transfers = []) {
      const amountInValue =
        userTrade?.amountIn?.text ||
        tradeDetails?.offerText ||
        totalAmount?.value ||
        ''
      const amountInToken =
        userTrade?.amountIn?.token ||
        tradeDetails?.offerToken ||
        this.extractContractAmountToken(amountInValue) ||
        this.inferContractTradeTokenFromTransfers(transfers, amountInValue)
      const amountIn = this.formatContractDisplayAmount(
        amountInValue,
        amountInToken
      )
      const subtext =
        tradeDetails?.offerText || userTrade?.amountIn?.text
          ? ''
          : totalAmount?.subtext || ''

      if (amountIn && amountIn !== 'Not exposed') {
        return {
          label: 'Amount In',
          value: amountIn,
          asset: amountInToken
            ? this.resolveContractFlowAsset(amountInToken)
            : '',
          ...(subtext ? { subtext } : {}),
        }
      }

      return {
        label: 'Amount In',
        value: 'Not exposed',
        subtext: 'No input amount found in attached funds or emitted events',
        isUnavailable: true,
      }
    },
    formatContractDisplayAmount(value, token = '') {
      if (!value) {
        return ''
      }

      const trimmed = `${value}`.trim()
      if (!trimmed) {
        return ''
      }

      const directMatch = trimmed.match(/^([0-9]+(?:\.[0-9]+)?)(?:\s+(.+))?$/)
      if (!directMatch) {
        return trimmed
      }

      const [, amountPart, tokenPart = ''] = directMatch
      const inferredToken = tokenPart || token
      const formattedAmount = this.formatEventAmount(amountPart)

      return inferredToken
        ? `${formattedAmount} ${inferredToken}`.trim()
        : formattedAmount
    },
    extractContractAmountToken(value) {
      if (!value) {
        return ''
      }

      const trimmed = `${value}`.trim()
      const match = trimmed.match(/^[0-9][0-9,]*(?:\.[0-9]+)?\s+(.+)$/)
      return match?.[1]?.trim() || ''
    },
    parseContractDisplayAmount(value) {
      if (!value) {
        return null
      }

      const trimmed = `${value}`.trim()
      const match = trimmed.match(/^([0-9][0-9,]*(?:\.[0-9]+)?)(?:\s+(.+))?$/)
      if (!match) {
        return null
      }

      return {
        amountValue: this.parseNumericString(match[1]),
        token: match[2]?.trim().toUpperCase() || '',
      }
    },
    prioritizeContractAssetFlow(transfers, amountOutStat, amountInStat) {
      const amountOut = this.parseContractDisplayAmount(amountOutStat?.value)
      const amountIn = this.parseContractDisplayAmount(amountInStat?.value)

      return (transfers ?? [])
        .map((transfer, index) => ({
          transfer,
          index,
          score: this.getContractAssetFlowPriorityScore(
            transfer,
            amountOut,
            amountIn
          ),
        }))
        .sort((a, b) => {
          if (a.score !== b.score) {
            return a.score - b.score
          }
          return a.index - b.index
        })
        .map((entry) => entry.transfer)
    },
    getContractAssetFlowPriorityScore(transfer, amountOut, amountIn) {
      const token = `${transfer?.token ?? ''}`.trim().toUpperCase()
      const amountValue = transfer?.amountValue || 0

      if (
        amountOut?.token &&
        token === amountOut.token &&
        Number.isFinite(amountOut.amountValue)
      ) {
        const diff = Math.abs(amountValue - amountOut.amountValue)
        if (diff < 1e-9) {
          return 0
        }

        return 10 + diff
      }

      if (
        amountIn?.token &&
        token === amountIn.token &&
        Number.isFinite(amountIn.amountValue)
      ) {
        const diff = Math.abs(amountValue - amountIn.amountValue)
        if (diff < 1e-9) {
          return 20
        }

        return 30 + diff
      }

      return 1000 - Math.min(amountValue, 999)
    },
    inferContractTradeTokenFromTransfers(transfers, amount, options = {}) {
      const excludedTokens = new Set(
        (options.excludeTokens ?? [])
          .map((token) => `${token}`.trim().toUpperCase())
          .filter(Boolean)
      )
      const normalizedTransfers = (transfers ?? []).filter(
        (transfer) =>
          transfer?.token && !excludedTokens.has(`${transfer.token}`.trim().toUpperCase())
      )
      if (normalizedTransfers.length === 0) {
        return ''
      }

      const numericAmount = this.parseNumericString(amount)
      if (Number.isFinite(numericAmount) && numericAmount > 0) {
        const directMatch = normalizedTransfers.find(
          (transfer) => Math.abs((transfer.amountValue || 0) - numericAmount) < 1e-9
        )
        if (directMatch?.token) {
          return directMatch.token
        }

        const groupedMatch = this.groupTransfersByToken(normalizedTransfers).find(
          (group) => Math.abs((group.total || 0) - numericAmount) < 1e-9
        )
        if (groupedMatch?.token) {
          return groupedMatch.token
        }
      }

      return (
        this.groupTransfersByToken(normalizedTransfers).sort(
          (a, b) => (b.total || 0) - (a.total || 0)
        )[0]?.token || ''
      )
    },
    getContractMethodDisplay(contract, events = []) {
      const specificEvent = `${this.getPrimaryContractEventType(events)}`.toLowerCase()
      if (specificEvent.includes('trade') || specificEvent.includes('swap')) {
        return 'Swap'
      }
      if (specificEvent.includes('strategy.execute')) {
        return 'Strategy Execute'
      }
      if (specificEvent.includes('process.reply')) {
        return 'Process Reply'
      }
      if (specificEvent.includes('process')) {
        return 'Strategy Process'
      }
      if (specificEvent.includes('deposit')) {
        return 'Deposit'
      }
      if (specificEvent.includes('withdraw')) {
        return 'Withdraw'
      }
      if (specificEvent.includes('unstake')) {
        return 'Unstake'
      }
      if (specificEvent.includes('stake')) {
        return 'Stake'
      }

      const candidates = [
        specificEvent,
        this.getContractMessageName(contract?.msg),
        contract?.attributes?.action,
        this.getContractEventAttribute(contract?.contractEvents, [
          'action',
          'method',
          'execute_msg',
          'message_type',
        ]),
        contract?.contractType,
      ]

      for (const candidate of candidates) {
        const formatted = this.formatContractMethodLabel(candidate)
        if (formatted && formatted !== 'Execute Contract') {
          return formatted
        }
      }

      return 'Execute Contract'
    },
    layoutContractExecutionDetails(items) {
      const compactItems = (items ?? []).filter(Boolean)
      const regularItems = compactItems.filter((item) => !item.fullWidth)
      const wideItems = compactItems.filter((item) => item.fullWidth)

      if (regularItems.length % 2 === 1) {
        const lastIndex = regularItems.length - 1
        regularItems[lastIndex] = {
          ...regularItems[lastIndex],
          fullWidth: true,
        }
      }

      return [...regularItems, ...wideItems]
    },
    extractContractUserTrade(address, transfers) {
      if (!address || !Array.isArray(transfers) || transfers.length === 0) {
        return null
      }

      const normalizedAddress = address.toLowerCase()
      const amountIn = this.getTopContractTransferForAddress(
        transfers,
        'sender',
        normalizedAddress
      )
      const amountOut = this.getTopContractTransferForAddress(
        transfers,
        'recipient',
        normalizedAddress
      )

      if (!amountIn && !amountOut) {
        return null
      }

      return {
        amountIn: amountIn
          ? {
              text: `${amountIn.amountText} ${amountIn.token}`.trim(),
              token: amountIn.token,
            }
          : null,
        amountOut: amountOut
          ? {
              text: `${amountOut.amountText} ${amountOut.token}`.trim(),
              token: amountOut.token,
            }
          : null,
      }
    },
    getTopContractTransferForAddress(transfers, key, address) {
      return (transfers ?? [])
        .filter(
          (transfer) => `${transfer?.[key] ?? ''}`.toLowerCase() === address
        )
        .sort((a, b) => (b.amountValue || 0) - (a.amountValue || 0))[0]
    },
    resolveContractFlowAsset(token) {
      if (!token) {
        return ''
      }

      const normalizedToken = `${token}`.trim().toUpperCase()
      const poolMatch = (this.pools ?? []).find((pool) => {
        const asset = assetFromString(pool.asset)
        if (!asset) {
          return false
        }

        const candidates = [
          `${asset.chain}.${asset.symbol}`.toUpperCase(),
          `${asset.symbol}`.toUpperCase(),
          `${asset.ticker}`.toUpperCase(),
          `${asset.symbol}`.replace(/\//g, '-').toUpperCase(),
          `${asset.symbol}`.replace(/-/g, '/').toUpperCase(),
        ]

        return candidates.includes(normalizedToken)
      })

      if (poolMatch?.asset) {
        return poolMatch.asset
      }

      if (normalizedToken === 'RUNE') {
        return 'THOR.RUNE'
      }

      if (normalizedToken === 'TCY') {
        return 'THOR.TCY'
      }

      return normalizedToken.includes('.')
        ? normalizedToken
        : `THOR.${normalizedToken.replace(/\//g, '-')}`
    },
    formatContractMethodLabel(value) {
      if (!value) {
        return ''
      }

      const cleaned = `${value}`
        .split('/')
        .filter(Boolean)
        .slice(-1)[0]
        .replace(/^cosmwasm\./i, '')
        .replace(/^wasm\./i, '')
        .replace(/(^|[.])v\d+(?=\.|$)/gi, '$1')
        .replace(/(^|[.])msg/gi, '$1')
        .replace(/([a-z0-9])([A-Z])/g, '$1 $2')
        .replace(/[._-]+/g, ' ')
        .replace(/\s+/g, ' ')
        .trim()

      if (!cleaned) {
        return ''
      }

      return cleaned
        .split(' ')
        .filter(Boolean)
        .map((segment) => segment.charAt(0).toUpperCase() + segment.slice(1))
        .join(' ')
    },
    extractThorchainNetworkFee(thorStatus) {
      const tx = thorStatus?.tx
      const feeCoin =
        tx?.fee?.amount?.[0] ||
        tx?.fee?.[0] ||
        tx?.gas?.[0] ||
        null

      if (!feeCoin) {
        return null
      }

      const described = this.describeContractCoin(feeCoin)
      if (!described) {
        return null
      }

      return {
        amountDisplay: described,
        payer: tx?.from_address || '',
      }
    },
    getContractMessageName(msg) {
      if (!msg) {
        return ''
      }

      if (typeof msg === 'string') {
        const trimmed = msg.trim()
        if (!trimmed) {
          return ''
        }

        try {
          return this.getContractMessageName(JSON.parse(trimmed))
        } catch (_error) {
          return trimmed.startsWith('{')
            ? ''
            : this.humanizeContractLabel(trimmed)
        }
      }

      if (Array.isArray(msg) || typeof msg !== 'object') {
        return ''
      }

      const [firstKey, firstValue] = Object.entries(msg)[0] ?? []
      if (!firstKey) {
        return ''
      }

      if (
        ['wasm', 'execute', 'msg'].includes(firstKey) &&
        firstValue &&
        typeof firstValue === 'object'
      ) {
        return this.getContractMessageName(firstValue)
      }

      return this.humanizeContractLabel(firstKey)
    },
    getContractAddress(contract) {
      return (
        contract?.attributes?.contract_address ||
        this.getContractEventAttribute(contract?.contractEvents, [
          '_contract_address',
          'contract_address',
        ]) ||
        ''
      )
    },
    getContractEventAttribute(events, keys) {
      if (!Array.isArray(events) || !Array.isArray(keys)) {
        return ''
      }

      const wanted = new Set(keys.map((key) => String(key).toLowerCase()))
      for (const event of events) {
        for (const attr of event?.attributes ?? []) {
          const key = String(attr?.key ?? '').toLowerCase()
          if (wanted.has(key) && attr?.value != null && `${attr.value}`.trim()) {
            return `${attr.value}`.trim()
          }
        }
      }

      return ''
    },
    getContractEventAttributes(event) {
      const attributes = {}
      for (const attr of event?.attributes ?? []) {
        const key = `${attr?.key ?? ''}`.trim()
        if (!key) {
          continue
        }
        attributes[key] = `${attr?.value ?? ''}`.trim()
      }
      return attributes
    },
    getRujiraContractMeta(address) {
      if (!address || !this.rujiraRegistry?.addresses) {
        return null
      }

      return this.rujiraRegistry.addresses[address.toLowerCase()] ?? null
    },
    buildContractParty(address) {
      if (!address) {
        return {
          text: '',
          address: '',
          name: '',
          shortAddress: '',
        }
      }

      const meta = this.getRujiraContractMeta(address)
      const shortAddress = this.formatAddress(address)

      return {
        text: meta ? `${meta.name} (${shortAddress})` : shortAddress,
        address,
        name: meta?.name || '',
        shortAddress,
      }
    },
    formatContractParty(address) {
      if (!address) {
        return ''
      }

      return this.buildContractParty(address).text
    },
    extractTouchedContracts(events) {
      if (!Array.isArray(events)) {
        return []
      }

      const touched = new Set()
      for (const event of events) {
        for (const attr of event?.attributes ?? []) {
          if (`${attr?.key ?? ''}`.toLowerCase() === '_contract_address') {
            const value = `${attr?.value ?? ''}`.trim()
            if (value) {
              touched.add(value)
            }
          }
        }
      }

      return [...touched]
    },
    extractContractTransfers(events) {
      if (!Array.isArray(events)) {
        return []
      }

      const transfers = events
        .filter((event) => event?.type === 'transfer')
        .map((event) => {
          const attrs = this.getContractEventAttributes(event)
          const amount = this.parseEventCoinAmount(attrs.amount)
          return {
            sender: attrs.sender || '',
            recipient: attrs.recipient || '',
            amountValue: amount?.value ?? 0,
            amountText: amount?.text ?? '',
            token: amount?.token ?? '',
          }
        })
        .filter((transfer) => transfer.amountText && transfer.token)

      const deduped = []
      const seen = new Set()
      for (const transfer of transfers) {
        const key = [
          transfer.amountText,
          transfer.token,
          transfer.sender,
          transfer.recipient,
        ].join('|')
        if (seen.has(key)) {
          continue
        }
        seen.add(key)
        deduped.push(transfer)
      }

      return deduped
        .filter((transfer, _, all) => {
          if (transfer.sender !== transfer.recipient) {
            return true
          }

          return !all.some(
            (other) =>
              other !== transfer &&
              other.amountText === transfer.amountText &&
              other.token === transfer.token &&
              other.sender !== other.recipient
          )
        })
        .sort((a, b) => (b.amountValue || 0) - (a.amountValue || 0))
    },
    extractContractNetworkFee(events) {
      if (!Array.isArray(events)) {
        return null
      }

      const txEvent = events.find((event) => event?.type === 'tx')
      const attrs = this.getContractEventAttributes(txEvent)
      const amount = this.parseEventCoinAmount(attrs.fee)
      if (!amount?.text || amount.text === '-') {
        return null
      }

      return {
        amountText: amount.text,
        amountDisplay: amount.token
          ? `${amount.text} ${amount.token}`.trim()
          : amount.text,
        payer: attrs.fee_payer || '',
      }
    },
    extractContractTrade(events, transfers) {
      if (!Array.isArray(events)) {
        return null
      }

      const tradeEvent = events.find(
        (event) => event?.type && `${event.type}`.toLowerCase().includes('trade')
      )
      if (!tradeEvent) {
        return null
      }

      const attrs = this.getContractEventAttributes(tradeEvent)
      const transferGroups = this.groupTransfersByToken(transfers)
      const offerValue = this.parseNumericString(attrs.offer)
      const bidValue = this.parseNumericString(attrs.bid)
      const offerMatch = this.findTransferMatchByAmount(transfers, transferGroups, offerValue)
      const bidMatch = this.findTransferMatchByAmount(transfers, transferGroups, bidValue)

      return {
        offerText: offerMatch
          ? `${offerMatch.amountText} ${offerMatch.token}`
          : attrs.offer || '',
        offerToken: offerMatch?.token || '',
        bidText: bidMatch
          ? `${bidMatch.amountText} ${bidMatch.token}`
          : attrs.bid || '',
        bidToken: bidMatch?.token || '',
        rate: attrs.rate || attrs.price || '',
      }
    },
    groupTransfersByToken(transfers) {
      const groups = {}
      for (const transfer of transfers ?? []) {
        if (!transfer?.token) {
          continue
        }
        if (!groups[transfer.token]) {
          groups[transfer.token] = {
            token: transfer.token,
            total: 0,
          }
        }
        groups[transfer.token].total += transfer.amountValue || 0
      }
      return Object.values(groups)
    },
    findTransferMatchByAmount(transfers, groups, amount) {
      if (!Number.isFinite(amount) || amount <= 0) {
        return null
      }

      const directMatch = (transfers ?? []).find(
        (transfer) => Math.abs((transfer.amountValue || 0) - amount) < 1e-9
      )
      if (directMatch) {
        return directMatch
      }

      const groupMatch = (groups ?? []).find(
        (group) => Math.abs((group.total || 0) - amount) < 1e-9
      )
      if (!groupMatch) {
        return null
      }

      return {
        amountText: this.formatEventAmount(groupMatch.total),
        token: groupMatch.token,
      }
    },
    describeContractTransfers(transfers) {
      if (!transfers?.length) {
        return ''
      }

      return this.joinHumanList(
        transfers.slice(0, 3).map((transfer) => {
          let text = `${transfer.amountText} ${transfer.token}`
          if (transfer.sender) {
            text += ` from ${this.formatContractParty(transfer.sender)}`
          }
          if (transfer.recipient) {
            text += ` to ${this.formatContractParty(transfer.recipient)}`
          }
          return text
        })
      )
    },
    getPrimaryContractEventType(events) {
      if (!Array.isArray(events)) {
        return ''
      }

      const ignored = new Set([
        'tx',
        'message',
        'execute',
        'reply',
        'coin_spent',
        'coin_received',
        'transfer',
      ])

      return (
        events.find((event) => !ignored.has(`${event?.type ?? ''}`.toLowerCase()))
          ?.type || ''
      )
    },
    parseEventCoinAmount(value) {
      if (!value) {
        return null
      }

      const match = `${value}`.trim().match(/^([0-9]+(?:\.[0-9]+)?)(.+)$/)
      if (!match) {
        return {
          value: this.parseNumericString(value),
          text: `${value}`.trim(),
          token: '',
        }
      }

      const [, amountPart, denomPart] = match
      return {
        value: this.parseNumericString(amountPart),
        text: this.formatEventAmount(amountPart),
        token: this.normalizeEventDenom(denomPart),
      }
    },
    parseNumericString(value) {
      const number = Number.parseFloat(`${value ?? ''}`.replace(/,/g, ''))
      return Number.isFinite(number) ? number : NaN
    },
    formatEventAmount(value) {
      const number = this.parseNumericString(value)
      if (!Number.isFinite(number)) {
        return `${value}`.trim()
      }

      return this.normalFormat(number)
    },
    normalizeEventDenom(denom) {
      if (!denom) {
        return ''
      }

      return `${denom}`
        .trim()
        .replace(/-0x[a-f0-9]+$/i, '')
        .replace(/^u([a-z])/i, '$1')
        .replace(/_/g, ' ')
        .toUpperCase()
    },
    describeContractFunds(funds, fallbackCoins = []) {
      const source =
        Array.isArray(funds) && funds.length > 0
          ? funds
          : Array.isArray(fallbackCoins)
            ? fallbackCoins
            : []

      const describedFunds = source
        .map((coin) => this.describeContractCoin(coin))
        .filter(Boolean)

      return this.joinHumanList(describedFunds.slice(0, 2))
    },
    describeContractCoin(coin) {
      if (!coin) {
        return ''
      }

      const parsedAsset = coin?.asset
        ? this.parseMemoAsset(coin.asset, this.pools)
        : null
      const assetLabel = parsedAsset
        ? this.showAsset(parsedAsset)
        : this.humanizeContractLabel(coin?.denom || coin?.asset || '')
      const amount = Number(coin?.amount)

      if (Number.isFinite(amount) && amount > 0) {
        if (parsedAsset) {
          return `${this.baseAmountFormatOrZero(amount)} ${assetLabel}`
        }
        return `${this.normalFormat(amount)} ${assetLabel}`.trim()
      }

      return assetLabel || ''
    },
    describeContractEvents(events) {
      if (!Array.isArray(events) || events.length === 0) {
        return ''
      }

      const eventTypes = [
        ...new Set(
          events
            .map((event) => this.humanizeContractLabel(event?.type))
            .filter(Boolean)
        ),
      ]

      if (eventTypes.length === 0) {
        return ''
      }

      const preview = this.joinHumanList(eventTypes.slice(0, 3))
      return `It emitted ${events.length} event${
        events.length === 1 ? '' : 's'
      }, including ${preview}.`
    },
    describeContractHighlights(contract) {
      const details = []
      const attributes = contract?.attributes ?? {}

      const amount = attributes?.amount
      if (amount != null && `${amount}`.trim()) {
        details.push(`amount ${this.normalFormat(amount)}`)
      }

      const shares = attributes?.shares
      if (shares != null && `${shares}`.trim()) {
        details.push(`shares ${this.normalFormat(shares)}`)
      }

      const recipient =
        attributes?.recipient ||
        attributes?.to ||
        this.getContractEventAttribute(contract?.contractEvents, [
          'recipient',
          'to',
          'to_address',
        ])
      if (recipient) {
        details.push(`recipient ${this.formatAddress(recipient)}`)
      }

      const sender =
        attributes?.sender ||
        attributes?.from ||
        this.getContractEventAttribute(contract?.contractEvents, [
          'sender',
          'from',
          'from_address',
        ])
      if (sender) {
        details.push(`sender ${this.formatAddress(sender)}`)
      }

      const action =
        attributes?.action ||
        this.getContractEventAttribute(contract?.contractEvents, ['action'])
      if (action) {
        details.push(`action ${this.humanizeContractLabel(action)}`)
      }

      return [...new Set(details)].slice(0, 3)
    },
    describeContractFailureReason(logs) {
      if (!logs) {
        return ''
      }

      const text = Array.isArray(logs) ? logs.join(' ') : `${logs}`
      return text.replace(/\s+/g, ' ').trim().slice(0, 180)
    },
    humanizeContractLabel(value) {
      if (value == null) {
        return ''
      }

      return `${value}`
        .split('/')
        .filter(Boolean)
        .slice(-1)[0]
        .replace(/([a-z0-9])([A-Z])/g, '$1 $2')
        .replace(/[_-]+/g, ' ')
        .replace(/\s+/g, ' ')
        .trim()
        .toLowerCase()
    },
    joinHumanList(items) {
      if (!items || items.length === 0) {
        return ''
      }
      if (items.length === 1) {
        return items[0]
      }
      if (items.length === 2) {
        return `${items[0]} and ${items[1]}`
      }
      return `${items.slice(0, -1).join(', ')}, and ${items[items.length - 1]}`
    },
    createLoanRepayment(thorStatus, actions, thorTx) {
      const action = actions.actions[0]

      const ins = action?.in.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        from: a?.address,
        done: true,
      }))

      let outs = []
      outs = action?.out.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        to: a?.address,
        height: a?.height,
        done: true,
      }))

      if (outs?.length === 0 && thorStatus.planned_out_txs?.length > 0) {
        thorStatus.planned_out_txs.map((t) => ({
          asset: this.parseMemoAsset(t.coin.asset),
          amount: t.coin.amount,
          to: t.to_address,
          done: false,
          pending: true,
        }))
      }

      return {
        cards: {
          title: 'Loan Repay',
          in: ins,
          middle: {
            pending: false,
          },
          out: outs,
        },
        accordions: {
          in: ins,
          action: {
            type: 'Repay',
            timeStamp: moment.unix(action?.date / 1e9) || null,
            height: action?.height,
            done: true,
          },
          out: outs,
        },
      }
    },
    createContractState(thorStatus, action) {
      const ins = action?.in.map((a) => ({
        type: 'Caller',
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        from: a?.address,
        height: action?.height,
        timestamp: moment.unix(action?.date / 1e9),
        done: true,
      }))

      const code = action.metadata?.contract?.code ?? 0
      const logs = action.metadata?.contract?.logs
      const memo = action.metadata?.contract?.memo
      const hasError = code > 0

      return {
        cards: {
          title: 'Contract Event',
          overview: this.buildContractOverview(action, thorStatus),
          summary: this.buildContractSummary(action),
          in: [
            {
              icon: require('@/assets/images/wallet.svg?inline'),
              address: action?.in[0]?.address,
            },
          ],
          middle: {
            pending: false,
            fail: hasError,
            success: !hasError,
            empty: true,
          },
          out: [
            {
              icon: require('@/assets/images/contract.svg?inline'),
              address: action?.out[0]?.address,
            },
          ],
        },
        accordions: {
          in: ins,
          action: {
            type: 'Contract Call',
            attributes: {
              attributes: action.metadata?.contract?.attributes,
              funds: action.metadata?.contract?.funds,
              msg: action.metadata?.contract?.msg,
            },
            memo,
            logs,
            code,
            error: hasError,
            reason: hasError ? logs : undefined,
            done: true,
          },
          events: action.metadata?.contract?.contractEvents,
          out: [],
        },
      }
    },
    createBondState(thorStatus, action, thorTx) {
      action = action.actions[0]
      const timeStamp = moment.unix(action?.date / 1e9)

      const ins = action?.in.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        gas: thorStatus?.tx?.gas ? thorStatus?.tx?.gas?.[0]?.amount : null,
        gasAsset: thorStatus?.tx?.gas
          ? this.parseMemoAsset(thorStatus?.tx?.gas?.[0]?.asset, this.pools)
          : null,
        txid: a?.txID,
        from: a?.address,
        done: true,
      }))

      const outs = action?.out.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        to: a?.address,
        done: true,
      }))

      const isWhitelist = action.metadata?.bond?.provider

      return {
        cards: {
          title: 'Bond' + (isWhitelist ? ' Whitelist' : ''),
          in: ins,
          middle: {
            pending: false,
          },
          out: [
            {
              icon: require('@/assets/images/node.svg?inline'),
              address: action.metadata?.bond?.nodeAddress,
              class: 'pad-icon',
            },
          ],
        },
        accordions: {
          in: ins,
          action: {
            type: 'Bond',
            memo: action.metadata?.bond?.memo,
            nodeAddress: action.metadata?.bond?.nodeAddress,
            provider: action.metadata?.bond?.provider,
            timeStamp,
            height: action?.height,
            done: true,
          },
          out: outs,
        },
      }
    },
    createUnbondState(thorStatus, action, thorTx) {
      action = action.actions[0]
      const timeStamp = moment.unix(action?.date / 1e9)

      const ins = action?.in.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        gas: thorStatus?.tx?.gas ? thorStatus?.tx?.gas?.[0]?.amount : null,
        gasAsset: thorStatus?.tx?.gas
          ? this.parseMemoAsset(thorStatus?.tx?.gas?.[0]?.asset, this.pools)
          : null,
        txid: a?.txID,
        from: a?.address,
        done: true,
      }))

      const outs = action?.out.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        to: a?.address,
        done: true,
      }))

      return {
        cards: {
          title: 'Unbond',
          in: [
            {
              icon: require('@/assets/images/node.svg?inline'),
              address: action.metadata?.bond?.nodeAddress,
              class: 'pad-icon',
            },
          ],
          middle: {
            pending: false,
          },
          out: outs,
        },
        accordions: {
          in: ins,
          action: {
            type: 'Unbond',
            memo: action.metadata?.bond?.memo,
            nodeAddress: action.metadata?.bond?.nodeAddress,
            timeStamp,
            height: action?.height,
            done: true,
          },
          out: outs,
        },
      }
    },
    createTradeDepositState(thorStatus, action, thorTx) {
      action = action.actions[0]
      const timeStamp = moment.unix(action?.date / 1e9)
      const memo = this.parseMemo(thorStatus?.tx?.memo)

      const ast = this.parseMemoAsset(
        thorStatus?.tx?.coins?.[0]?.asset,
        this.pools
      )

      let isSecure = false
      if (memo?.type?.includes('secure')) {
        isSecure = true
      }

      let error = false
      let reason = ''
      if (action.type === 'refund') {
        error = true
        reason = action.metadata?.refund?.reason
      }

      const ins = [
        {
          asset: ast,
          amount: thorStatus?.tx?.coins?.[0]?.amount ?? 0,
          txid: thorStatus?.tx?.id,
          from: thorStatus?.tx?.from_address,
          gas: thorStatus?.tx?.gas ? thorStatus?.tx?.gas?.[0]?.amount : null,
          gasAsset: thorStatus?.tx?.gas
            ? this.parseMemoAsset(thorStatus?.tx?.gas?.[0]?.asset, this.pools)
            : null,
          done: true,
        },
      ]

      const outs = [
        {
          asset: isSecure ? assetToSecure(ast) : assetToTrade(ast),
          amount: thorStatus.out_txs
            ? thorStatus.out_txs[0]?.coins?.[0]?.amount
            : thorStatus?.tx?.coins?.[0]?.amount ?? 0,
          txid: thorStatus.out_txs ? thorStatus.out_txs[0]?.id : null,
          to: memo.address,
          done: true,
        },
      ]

      return {
        cards: {
          title: this.camelCase(memo.type),
          in: ins,
          middle: {
            fail: error,
            pending: false,
          },
          out: error ? [] : outs,
        },
        accordions: {
          in: ins,
          action: {
            type: 'Deposit',
            memo: thorStatus?.tx?.memo,
            height: action?.height,
            timeStamp,
            done: true,
            error,
            reason,
          },
          out: error ? [] : outs,
        },
      }
    },
    createTradeWithdrawState(thorStatus, action, thorTx) {
      action = action.actions[0]
      const timeStamp = moment.unix(action?.date / 1e9)
      const memo = this.parseMemo(thorStatus?.tx?.memo)

      const ast = this.parseMemoAsset(
        thorStatus?.tx?.coins?.[0]?.asset,
        this.pools
      )

      let isSecure = false
      if (memo?.type?.includes('secure')) {
        isSecure = true
      }

      const ins = [
        {
          asset: isSecure ? assetToSecure(ast) : assetToTrade(ast),
          amount: thorStatus?.tx?.coins?.[0]?.amount ?? 0,
          txid: thorStatus?.tx?.id,
          from: thorStatus?.tx?.from_address,
          gas: thorStatus?.tx?.gas ? thorStatus?.tx?.gas?.[0]?.amount : null,
          gasAsset: thorStatus?.tx?.gas
            ? this.parseMemoAsset(thorStatus?.tx?.gas?.[0]?.asset, this.pools)
            : null,
          done: true,
        },
      ]

      const outs = [
        {
          asset: isSecure ? securedToAsset(ast) : tradeToAsset(ast),
          amount: thorStatus.out_txs
            ? thorStatus.out_txs[0]?.coins?.[0]?.amount
            : thorStatus?.tx?.coins?.[0]?.amount ?? 0,
          txid: thorStatus.out_txs ? thorStatus.out_txs[0]?.id : null,
          to: memo.address,
          gas: thorStatus.out_txs
            ? thorStatus.out_txs[0]?.gas?.[0]?.amount
            : null,
          gasAsset: thorStatus.out_txs
            ? this.parseMemoAsset(
                thorStatus.out_txs[0]?.gas?.[0]?.asset,
                this.pools
              )
            : null,
          outboundSigned:
            thorStatus?.stages.outbound_signed?.completed ?? false,
          outboundETA:
            thorStatus?.stages.outbound_signed?.scheduled_outbound_height -
            this.thorHeight,
          done: thorStatus?.stages?.outbound_signed?.completed === true,
        },
      ]

      return {
        cards: {
          title: this.camelCase(memo.type),
          in: ins,
          middle: {
            pending: this.isTxInPending(thorStatus, action),
          },
          out: outs,
        },
        accordions: {
          in: ins,
          action: {
            type: 'Withdraw',
            memo: thorStatus?.tx?.memo,
            height: action?.height,
            timeStamp,
            done: true,
          },
          out: outs,
        },
      }
    },
    createTCYUnstake(thorStatus, actions, thorTx) {
      const TCYUnstake = actions?.actions?.find((a) => a.type === 'tcy_unstake')
      const RefundAction = actions?.actions?.find((a) => a.type === 'refund')

      let memo = TCYUnstake?.metadata?.tcy?.memo
      let reason = ''
      let outs = []
      let ins = []

      let isRefund = false
      if (RefundAction) {
        const m = Object.keys(RefundAction.metadata)[0]
        memo = RefundAction.metadata[m]?.memo ?? undefined
        reason =
          RefundAction.metadata[m]?.reason ??
          RefundAction.metadata[m]?.code ??
          undefined
        isRefund = true
      }

      let action = TCYUnstake
      if (isRefund) {
        action = RefundAction
      }

      if (isRefund === false) {
        outs = [
          {
            asset: TCYUnstake.out?.[0]?.coins?.[0]?.asset,
            amount: TCYUnstake.out?.[0]?.coins?.[0]?.amount,
            txid: TCYUnstake.out[0].txID,
            to: TCYUnstake.out[0].address,
            done: true,
          },
        ]

        ins = [
          {
            from: TCYUnstake.out[0].address,
            txid: TCYUnstake.out[0].txID,
            done: true,
          },
        ]
      }

      return {
        cards: {
          title: 'Unstake ' + (isRefund ? '(Refund)' : ''),
          in: [
            {
              icon: require('@/assets/images/vault.svg?inline'),
              text: 'Stake Module',
              class: 'pad-icon',
            },
          ],
          middle: {
            fail: isRefund,
            pending: false,
          },
          out: outs,
        },
        accordions: {
          in: ins,
          action: {
            type: 'Unstake',
            timeStamp: moment.unix(action?.date / 1e9) || null,
            height: action?.height,
            memo,
            reason,
            done: true,
          },
          out: outs,
        },
      }
    },
    createTCYStake(thorStatus, actions, thorTx) {
      const TCYStake = actions?.actions?.find((a) => a.type === 'tcy_stake')
      const RefundAction = actions?.actions?.find((a) => a.type === 'refund')

      let memo = TCYStake?.metadata?.tcy?.memo
      let reason = ''
      let outs = []
      let ins = []

      let isRefund = false
      if (RefundAction) {
        const m = Object.keys(RefundAction.metadata)[0]
        memo = RefundAction.metadata[m]?.memo ?? undefined
        reason =
          RefundAction.metadata[m]?.reason ??
          RefundAction.metadata[m]?.code ??
          undefined
        isRefund = true
      }

      let action = TCYStake
      if (isRefund) {
        action = RefundAction
      }

      if (isRefund === false) {
        ins = [
          {
            asset: TCYStake.in?.[0]?.coins?.[0]?.asset,
            amount: TCYStake.in?.[0]?.coins?.[0]?.amount,
            txid: TCYStake.in[0].txID,
            from: TCYStake.in[0].address,
            done: true,
          },
        ]

        outs = [
          {
            to: TCYStake.in[0].address,
            txid: TCYStake.in[0].txID,
            done: true,
          },
        ]
      }

      return {
        cards: {
          title: 'Stake ' + (isRefund ? '(Refund)' : ''),
          in: ins,
          middle: {
            fail: isRefund,
            pending: false,
          },
          out: [
            {
              icon: require('@/assets/images/vault.svg?inline'),
              text: 'Stake Module',
              class: 'pad-icon',
            },
          ],
        },
        accordions: {
          in: ins,
          action: {
            type: 'Stake',
            timeStamp: moment.unix(action?.date / 1e9) || null,
            height: action?.height,
            memo,
            reason,
            done: true,
          },
          out: outs,
        },
      }
    },
    createAbstractState(thorStatus, action, thorTx) {
      let ins = action?.in.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        from: a?.address,
        done: true,
      }))

      let outs = action?.out.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        to: a?.address,
        done: true,
      })) ?? []
      // Trade/secure only: when multiple outbounds have same asset and amount, only show one
      const hasTradeOrSecureOut = outs.some(
        (o) => o?.asset?.trade || o?.asset?.secure
      )
      if (hasTradeOrSecureOut) {
        const outKey = (o) =>
          `${o?.asset ? assetToString(o.asset) : ''}:${o?.amount ?? ''}`
        const seenOuts = new Set()
        outs = outs.filter((o) => {
          const key = outKey(o)
          if (seenOuts.has(key)) return false
          seenOuts.add(key)
          return true
        })
      }

      let memo
      let reason
      let isRefund = false
      if (action.metadata) {
        const m = Object.keys(action.metadata)[0]
        memo = action.metadata[m]?.memo ?? undefined
        reason =
          action.metadata[m]?.reason ?? action.metadata[m]?.code ?? undefined
        if (Object.keys(action.metadata).length === 1) {
          isRefund = m === 'refund'
        }
      }

      let cardAction = {
        type: action?.type === 'refund' ? 'Refund' : 'Action',
        timeStamp: moment.unix(action?.date / 1e9) || null,
        height: action?.height,
        memo,
        reason,
        done: true,
      }

      function kebabToTitle(kebab) {
        if (!kebab || typeof kebab !== 'string') return ''
        return kebab
          .split('_')
          .map(
            (word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
          )
          .join(' ')
      }

      let title = kebabToTitle(action.type) ?? 'Action'
      if (action.type === 'tcy_claim') {
        title = 'Tcy Claim'

        ins = [
          {
            icon: require('@/assets/images/vault.svg?inline'),
            text: 'Claim Module',
            class: 'pad-icon',
            from: action?.in[0].address,
            txid: action?.in[0].txID,
            done: true,
          },
        ]
      }

      if (action.type === 'limit_swap') {
        let ttl = 43200
        let swapLimit = 'N/A'
        if (memo.split(':')[3].split('/').length > 0) {
          ttl = memo.split(':')[3].split('/')[1] ?? 43200
          swapLimit = memo.split(':')[3].split('/')[0]
        }

        cardAction = {
          type: 'Order',
          timeStamp: moment.unix(action?.date / 1e9) || null,
          height: action?.height,
          memo,
          ttl,
          swapLimit,
          done: true,
        }

        outs[0] = { ...outs[0], hide: true }
      }

      return {
        cards: {
          title,
          in: ins,
          middle: {
            fail: isRefund,
            refund: isRefund,
            pending: false,
          },
          out: outs,
        },
        accordions: {
          in: ins,
          action: cardAction,
          out: outs,
        },
      }
    },
    async getOtherActionHash(actions, thorStatus) {
      let hash = thorStatus?.tx?.id

      hash = actions.actions
        ?.reduce(
          (r, a) => [
            ...a.in.map((i) => i.txID),
            ...a.out.map((o) => o.txID),
            ...r,
          ],
          []
        )
        .find((a) => a !== hash)

      const ts = (
        await this.$api.getTxStatus(hash).catch((e) => {
          if (e?.response?.status / 200 !== 1) {
            this.error.message =
              "Can't find transaction status. Please make sure the correct transaction hash or account address is inserted."
          }
        })
      )?.data

      return ts
    },
    createNativeTx(nativeTx) {
      const inAsset = nativeTx?.in?.[0]?.coins?.[0]?.asset
      const inAmount = nativeTx?.in?.[0]?.coins?.[0]?.amount
      const timeStamp = moment(nativeTx.date / 1e6)

      const cards = {
        title: 'Send',
        in: [
          {
            asset: inAsset,
            amount: inAmount,
          },
        ],
        middle: {
          send: true,
        },
        out: [
          {
            icon: require('@/assets/images/wallet.svg?inline'),
            address: nativeTx?.out[0]?.address,
          },
        ],
      }

      const isError = nativeTx?.metadata?.send?.code !== '0'

      const accordions = {
        in: [],
        action: {
          type: 'send',
          txid: nativeTx?.in[0]?.txID,
          memo: nativeTx.metadata?.send?.memo || '',
          from: nativeTx?.in[0]?.address,
          to: nativeTx?.out[0]?.address,
          height: nativeTx?.height,
          gas: nativeTx?.metadata?.send?.networkFees?.[0]?.amount,
          gasAsset: 'THOR.RUNE',
          timeStamp,
          pending: false,
          error: isError,
          reason: isError ? nativeTx?.metadata?.send?.reason : undefined,
          done: true,
          showAtFirst: true,
        },
        out: [],
      }

      this.$set(this, 'cards', [this.createCard(cards, accordions)])
    },
    createAddLiquidityState(thorStatus, actions, thorTx, memo) {
      const isSaver = this.parseMemoAsset(memo?.asset)?.synth

      const inAsset = this.parseMemoAsset(
        thorStatus?.tx?.coins?.[0]?.asset,
        this.pools
      )
      const inAmount = parseInt(thorStatus?.tx?.coins?.[0]?.amount ?? 0)
      const addAction = actions?.actions?.find(
        (a) => a.type === 'addLiquidity' && a.in?.[0]?.address !== ''
      )
      const timeStamp = moment.unix(addAction?.date / 1e9)

      const isRefund = actions?.actions?.find((a) => a.type === 'refund')

      const outboundDelayRemaining =
        (thorStatus?.stages.outbound_delay?.remaining_delay_seconds ?? 0) ||
        (thorStatus?.stages.outbound_delay?.remaining_delay_blocks ?? 0) *
          this.blockSeconds('THOR')

      const pending =
        thorStatus?.stages.swap_status?.pending ||
        !thorStatus?.stages.inbound_observed?.completed ||
        !(thorStatus?.stages.inbound_confirmation_counted?.completed ?? true) ||
        !thorStatus?.stages.inbound_finalised?.completed ||
        (inAsset?.chain === 'THOR' && addAction.status === 'pending')

      const memoText =
        thorStatus?.tx?.memo || isRefund?.metadata?.refund?.memo

      return {
        cards: {
          title: 'add Liquidity' + (isRefund ? ' (Refunded)' : ''),
          in: [
            {
              asset: inAsset,
              amount: inAmount,
            },
          ],
          middle: {
            pending,
            fail: isRefund,
          },
          out: [
            {
              text: isSaver ? 'THORChain Vault' : 'THORChain Pool',
              icon: require('@/assets/images/safe.svg?inline'),
              borderColor: 'var(--primary-color)',
            },
          ],
        },
        accordions: {
          in: [
            {
              txid: thorStatus?.tx?.id,
              from: thorStatus?.tx?.from_address,
              asset: inAsset,
              amount: inAmount,
              done: true,
            },
          ],
          action: {
            type: isRefund ? 'Refund' : 'Add',
            timeStamp: timeStamp || null,
            liquidityUnits:
              parseInt(addAction?.metadata?.addLiquidity?.liquidityUnits) ||
              null,
            affiliateName: memo.affiliate,
            affiliateFee: sumAffiliateFee(memo.fee),
            outboundDelayRemaining: outboundDelayRemaining || 0,
            outboundETA:
              thorStatus?.stages.outbound_signed?.scheduled_outbound_height -
              this.thorHeight,
            outboundSigned:
              thorStatus?.stages.outbound_signed?.completed ?? false,
            refundReason: isRefund
              ? isRefund?.metadata?.refund?.reason
              : undefined,
            memo: memoText,
            done:
              !thorStatus?.stages.swap_status?.pending &&
              !(inAsset?.chain === 'THOR' && addAction.status === 'pending'),
          },
          out: [],
        },
      }
    },
    getInboundStages(inbound) {
      const ret = []

      if (inbound?.done) {
        ret.push({
          text: 'done',
        })
      }

      if (inbound?.inboundConfCount) {
        ret.push({
          text: 'Confirm Counted',
        })
      }

      if (inbound?.observationsCompleted) {
        ret.push({
          text: 'Observed',
        })
      }

      return ret
    },
    getOutboundStages(outbound) {
      const ret = []

      if (outbound?.done) {
        ret.push({
          text: 'done',
        })
      }

      if (outbound?.outboundDelayRemaining) {
        ret.push({
          text: 'delayed',
          class: 'yellow',
        })
      }

      if (outbound?.outboundSigned) {
        ret.push({
          text: 'signed',
        })
      } else if (outbound?.outboundSigned !== undefined) {
        ret.push({
          text: 'not signed',
          class: 'yellow',
        })
      }

      return ret
    },
    createRemoveLiquidityState(thorStatus, actions, thorTx, memo) {
      const inAsset = this.parseMemoAsset(
        thorStatus?.tx?.coins?.[0]?.asset,
        this.pools
      )
      const inAmount = parseInt(thorStatus?.tx?.coins?.[0]?.amount ?? 0)
      const withdrawAction = actions?.actions?.find(
        (a) => a.type === 'withdraw'
      )

      const refundAction = actions?.actions?.find((a) => a.type === 'refund')

      const outboundFees =
        withdrawAction?.metadata.withdraw?.networkFees.map((n) => n?.amount) ??
        []
      const outboundFeeAssets =
        outboundFees?.length > 0
          ? this.parseMemoAsset(
              withdrawAction?.metadata.withdraw?.networkFees.map(
                (n) => n?.asset
              ),
              this.pools
            )
          : []
      const timeStamp = moment.unix(
        (withdrawAction || refundAction)?.date / 1e9
      )

      const outTxs = thorStatus?.out_txs ?? undefined
      const userTxs =
        outTxs && new Set([outTxs.map((t) => t?.id?.toUpperCase())])

      let hasOngoing = false
      if (thorStatus.planned_out_txs > 0) {
        hasOngoing = thorStatus.planned_out_txs?.some(
          (tx) => !userTxs.has(tx.to_address.toUpperCase())
        )
        outTxs.push(
          thorStatus.planned_out_txs?.filter(
            (tx) => !userTxs.has(tx.to_address.toUpperCase())
          )
        )
      }

      let outAsset
      let outAmount
      const isOut = outTxs && outTxs[0]
      if (isOut) {
        outAsset = this.parseMemoAsset(
          outTxs[0]?.coins?.[0]?.asset,
          this.pools
        )
        outAmount =
          outTxs?.length > 0 ? parseInt(outTxs[0]?.coins?.[0]?.amount ?? 0) : 0
      }

      const outs = []
      if (outAsset) {
        outs.push({
          asset: outAsset,
          amount: outAmount,
        })
      }

      const moreOuts = outTxs?.slice(1)
      if (moreOuts && moreOuts.length > 0) {
        outs.push(
          ...moreOuts.map((o) => ({
            asset: this.parseMemoAsset(o.coins?.[0]?.asset, this.pools),
            amount: parseInt(o.coins?.[0]?.amount ?? 0),
          }))
        )
      }

      const outboundDelayRemaining =
        (thorStatus?.stages.outbound_delay?.remaining_delay_seconds ?? 0) ||
        (thorStatus?.stages.outbound_delay?.remaining_delay_blocks ?? 0) *
          this.blockSeconds('THOR')

      const outboundETA =
        this.thorHeight <
        thorStatus?.stages.outbound_signed?.scheduled_outbound_height
          ? thorStatus?.stages.outbound_signed?.scheduled_outbound_height -
            this.thorHeight
          : 0

      const outActions = []
      if (isOut) {
        outActions.push({
          fees: outboundFees,
          feeAssets: outboundFeeAssets,
          outboundDelayRemaining: outboundDelayRemaining || 0,
          outboundETA,
          outboundSigned:
            thorStatus?.stages.outbound_signed?.completed ?? false,
          done:
            thorStatus?.stages.outbound_signed?.completed ||
            outAsset?.chain === 'THOR',
        })

        if (moreOuts && moreOuts.length > 0) {
          outActions.push(
            ...moreOuts.map((o) => ({
              txid: o.id,
              to: o.to_address,
              asset: this.parseMemoAsset(o.coins?.[0]?.asset, this.pools),
              amount: parseInt(o.coins?.[0]?.amount ?? 0),
              gas: o.gas ? o.gas?.[0]?.amount : null,
              gasAsset: o.gas
                ? this.parseMemoAsset(o.gas?.[0]?.asset, this.pools)
                : null,
              outboundSigned:
                thorStatus?.stages.outbound_signed?.completed ?? false,
              done:
                thorStatus?.stages.outbound_signed?.completed ||
                outAsset?.chain === 'THOR',
            }))
          )
        }
      }

      let refundReason
      if (refundAction) {
        refundReason = refundAction?.metadata?.refund?.reason
      }

      return {
        cards: {
          title: 'Withdraw Liquidity',
          in: [
            {
              asset: inAsset,
              amount: inAmount,
            },
          ],
          middle: {
            pending: this.isTxInPending(thorStatus),
            fail: refundAction,
          },
          out: outs,
        },
        accordions: {
          in: [
            {
              txid: thorStatus?.tx.id,
              from: thorStatus?.tx.from_address,
              asset: inAsset,
              amount: inAmount,
              done: true,
            },
          ],
          action: {
            type: 'Withdraw',
            timeStamp: timeStamp || null,
            liquidityUnits:
              parseInt(withdrawAction?.metadata?.withdraw?.liquidityUnits) ||
              null,
            refundReason,
            done: !hasOngoing,
          },
          out: outActions,
        },
      }
    },
    createRunePoolDeposit(thorStatus, actions, thorTx, memo) {
      const action = actions.actions.find((a) => a.type === 'runePoolDeposit')

      const ins = action?.in.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        from: a?.address,
        done: true,
      }))

      const outs = action?.out.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        to: a?.address,
        done: true,
      }))

      return {
        cards: {
          title: 'RUNEPool Deposit',
          in: ins,
          middle: {
            pending: false,
          },
          out: [
            {
              text: 'THORChain Vault',
              icon: require('@/assets/images/safe.svg?inline'),
              borderColor: 'var(--primary-color)',
            },
          ],
        },
        accordions: {
          in: ins,
          action: {
            type: 'RUNEPool Deposit',
            timeStamp: moment.unix(action?.date / 1e9) || null,
            height: action?.height,
            units: parseInt(action?.metadata?.runePoolDeposit?.units) || null,
            done: true,
          },
          out: outs,
        },
      }
    },
    createRunePoolWithdraw(thorStatus, actions, thorTx, memo) {
      const action = actions.actions.find((a) => a.type === 'runePoolWithdraw')

      const ins = action?.in.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        from: a?.address,
        done: true,
      }))

      const outs = action?.out.map((a) => ({
        asset: this.parseMemoAsset(a.coins?.[0]?.asset),
        amount: a.coins?.[0]?.amount ?? 0,
        txid: a?.txID,
        to: a?.address,
        done: true,
      }))

      return {
        cards: {
          title: 'RUNEPool Withdraw',
          in: ins,
          middle: {
            pending: false,
          },
          out: outs,
        },
        accordions: {
          in: ins,
          action: {
            type: 'Withdraw',
            timeStamp: moment.unix(action?.date / 1e9) || null,
            height: action?.height,
            units: parseInt(action?.metadata?.runePoolWithdraw?.units) || null,
            done: true,
          },
          out: outs,
        },
      }
    },
    createSwapState(thorStatus, thorTx, actions, memo, thorHeader, quote) {
      // swap user addresses
      const userAddresses = new Set([
        thorStatus?.tx.from_address.toLowerCase(),
        memo.destAddr?.toLowerCase(), // TODO: sometimes the memo destAddr will be THORName
      ])
      // Non affiliate outs
      const memoAssetStr = (() => {
        const parsed = this.parseMemoAsset(memo?.asset)
        return parsed ? assetToString(parsed) : null
      })()
      let outTxs = thorStatus.out_txs?.filter(
        (tx) =>
          userAddresses.has(tx.to_address?.toLowerCase()) ||
          (tx.coins?.[0]?.asset === memoAssetStr &&
            tx.id !==
              '0000000000000000000000000000000000000000000000000000000000000000' &&
            tx.id !== '')
      )
      // get affiliate out if available
      const affiliateOut = thorStatus.out_txs?.filter(
        (tx) =>
          !userAddresses.has(tx.to_address?.toLowerCase()) &&
          (tx.id !==
            '0000000000000000000000000000000000000000000000000000000000000000' ||
            tx.id !== '')
      )
      // TODO: fix this in track code
      if (
        !outTxs ||
        outTxs?.length === 0 ||
        outTxs.every((o) => o.to_address === thorStatus?.tx.from_address) // Add scheduled outbound while having a refund
      ) {
        outTxs = thorStatus.planned_out_txs
          ?.filter((tx) => userAddresses.has(tx.to_address.toLowerCase()))
          .map((tx) => ({
            ...tx,
            coins: [{ amount: tx.coin.amount, asset: tx.coin.asset }],
          }))
      }

      // order by target swapped asset if we have refund in swap
      outTxs = orderBy(
        outTxs,
        (o) =>
          o.coins?.[0]?.asset === thorStatus?.tx?.coins?.[0]?.asset
      )

      // Trade/secure asset swap only: when multiple outbounds have same asset and amount, only show one
      const memoOutAsset = this.parseMemoAsset(memo?.asset, this.pools)
      if (memoOutAsset?.trade || memoOutAsset?.secure) {
        const outboundKey = (o) =>
          `${o.coins?.[0]?.asset ?? ''}:${o.coins?.[0]?.amount ?? ''}`
        const seenOut = new Set()
        outTxs = outTxs.filter((o) => {
          const key = outboundKey(o)
          if (seenOut.has(key)) return false
          seenOut.add(key)
          return true
        })
      }

      // Add native in/out search
      const inAsset = this.parseMemoAsset(
        thorStatus?.tx?.coins?.[0]?.asset,
        this.pools
      )
      const inAmount = parseInt(thorStatus?.tx?.coins?.[0]?.amount ?? 0)

      const outAsset = this.parseMemoAsset(
        outTxs?.length > 0 ? outTxs[0]?.coins?.[0]?.asset : memo?.asset,
        this.pools
      )
      let outAmount =
        outTxs?.length > 0 ? parseInt(outTxs[0]?.coins?.[0]?.amount ?? 0) : 0
      if (
        !outAmount &&
        actions?.actions?.length > 0 &&
        (outAsset?.trade || outAsset?.secure)
      ) {
        const outAssetStr = outAsset ? assetToString(outAsset) : null
        outAmount = parseInt(
          Object.values(
            groupBy(
              actions?.actions
                ?.find((a) => a.type === 'swap')
                ?.out?.filter(
                  (a) =>
                    a.coins?.[0]?.asset === outAssetStr
                ),
              'txID'
            )
          ).map((group) =>
            sumBy(group, (item) => +(item.coins?.[0]?.amount ?? 0))
          )[0]
        )
      }

      const outMemoAsset = this.parseMemoAsset(memo?.asset)

      // Midgard
      // There are multiple outbound fee
      // also there might be refund involved
      const swapAction = actions?.actions?.find((a) => a.type === 'swap')
      const outboundFees =
        swapAction?.metadata.swap?.networkFees.map((n) => n?.amount) ?? []
      const outboundFeeAssets =
        outboundFees?.length > 0
          ? this.parseMemoAsset(
              swapAction?.metadata.swap?.networkFees.map((n) => n?.asset),
              this.pools
            )
          : null
      let timeStamp = swapAction?.date
      const height = swapAction?.height
      this.height = height

      // Refunds
      const outboundHasRefund = outTxs?.some(
        (tx) => tx.refund || tx.memo?.toLowerCase().startsWith('refund')
      )
      // sometimes the outbound doesn't come out if the outbound is in native chain
      const outboundHasSuccess = outTxs?.some((tx) =>
        tx.memo?.toLowerCase().startsWith('out')
      )

      const inAmountUSD =
        (+swapAction?.metadata.swap.inPriceUSD * inAmount) / 1e8
      let outAmountUSD =
        (+swapAction?.metadata.swap.outPriceUSD *
          (outAmount || +this.quote?.expected_amount_out)) /
        1e8
      if (!outboundHasSuccess && outboundHasRefund) {
        outAmountUSD = (+swapAction?.metadata.swap.inPriceUSD * outAmount) / 1e8
      }

      const outboundRefundReason = actions?.actions.find(
        (action) => action.type === 'refund'
      )?.metadata?.refund?.reason

      // only refund happened
      const onlyRefund =
        actions?.actions.length > 0 &&
        actions?.actions.every((action) => action?.type === 'refund')

      const refundAction = actions?.actions?.find((a) => a.type === 'refund')
      if (onlyRefund) {
        timeStamp = refundAction?.date
      }
      let isRefund = false
      if (refundAction) {
        isRefund = true
      }

      // TODO: add nice check with animation (transition from pending to complete)
      // TODO: add failed swaps from midgard
      // TODO: Add refunded swap info
      // TODO: fix the loading check/spinner on complete
      // TODO: fix streaming card when finished
      // TODO: sometimes the pools price is fetched after the status

      const outboundDelayRemaining =
        (thorStatus?.stages.outbound_delay?.remaining_delay_seconds ?? 0) ||
        (thorStatus?.stages.outbound_delay?.remaining_delay_blocks ?? 0) *
          this.blockSeconds('THOR')

      if (timeStamp) {
        timeStamp = moment.unix(timeStamp / 1e9)
      }

      const rates = []
      if (inAmount && outAmount) {
        rates.push(
          `1 ${this.showTicker(inAsset)} = ${outAmount / inAmount} ${this.showTicker(outAsset)}`
        )
        rates.push(
          `1 ${this.showTicker(outAsset)} = ${inAmount / outAmount} ${this.showTicker(inAsset)}`
        )
      }

      return {
        cards: {
          title: onlyRefund ? 'refunded Swap' : 'swap',
          labels: onlyRefund ? [] : isRefund ? ['Refund'] : [],
          in: [
            {
              asset: inAsset,
              amount: inAmount,
              amountUSD: inAmountUSD,
            },
          ],
          middle: {
            pending: this.isTxInPending(thorStatus, actions),
            fail: onlyRefund,
          },
          out: [
            {
              asset: outAsset,
              amount: outAmount || +this.quote?.expected_amount_out,
              amountUSD: outAmountUSD,
              filter: outAmount
                ? undefined
                : (v) => `~ ${this.baseAmountFormatOrZero(v)}`,
            },
            ...(outTxs ?? []).slice(1).map((o) => {
              const oAmount = parseInt(o.coins?.[0]?.amount ?? 0)
              const isRefundTx =
                o.refund || o.memo?.toLowerCase().startsWith('refund')
              const priceUSD = isRefundTx
                ? +swapAction?.metadata?.swap?.inPriceUSD
                : +swapAction?.metadata?.swap?.outPriceUSD
              return {
                asset: this.parseMemoAsset(o.coins?.[0]?.asset, this.pools),
                amount: oAmount,
                amountUSD: (priceUSD * oAmount) / 1e8,
              }
            }),
          ],
        },
        accordions: {
          in: [
            {
              txid: thorStatus?.tx.id,
              from: thorStatus?.tx.from_address,
              asset: inAsset,
              amount: inAmount,
              gas: thorStatus?.tx.gas ? thorStatus?.tx?.gas?.[0]?.amount : null,
              gasAsset: thorStatus?.tx.gas
                ? this.parseMemoAsset(thorStatus?.tx?.gas?.[0]?.asset, this.pools)
                : null,
              preObservations:
                thorStatus?.stages?.inbound_observed?.pre_confirmation_count,
              observations: thorStatus?.stages?.inbound_observed?.final_count,
              observationsCompleted:
                thorStatus?.stages?.inbound_observed?.completed,
              finalisedHeight: thorTx.finalised_height,
              inboundObserved:
                thorStatus?.stages?.inbound_observed?.completed || false,
              inboundConfCount:
                thorStatus?.stages?.inbound_confirmation_counted || 0,
              preConfirmationCount:
                thorStatus?.stages?.inbound_observed?.pre_confirmation_count ||
                0,
              confirmationRemainingSeconds:
                thorStatus?.stages?.inbound_confirmation_counted
                  ?.remaining_confirmation_seconds || 0,
              done: thorStatus?.stages?.inbound_finalised?.completed,
            },
          ],
          action: {
            type: onlyRefund || isRefund ? 'refunded Swap' : 'swap',
            timeStamp: timeStamp || null,
            limit: memo?.limit,
            limitAsset: outMemoAsset,
            affiliateName: memo?.affiliate,
            affiliateFee: sumAffiliateFee(memo?.fee || 0),
            liquidityFee:
              parseInt(swapAction?.metadata.swap?.liquidityFee) || null,
            liquidityUnits: null,
            refundReason: onlyRefund || isRefund ? outboundRefundReason : null,
            asymmetry: null,
            affiliateOut: affiliateOut || undefined,
            swapSlip: parseInt(swapAction?.metadata.swap?.swapSlip),
            height,
            rate: rates,
            streaming: {
              count: thorStatus?.stages.swap_status?.streaming?.count,
              interval:
                thorStatus?.stages.swap_status?.streaming?.interval ||
                memo?.interval,
              quantity:
                thorStatus?.stages.swap_status?.streaming?.quantity ||
                memo?.quantity,
              lastHeight: null, // Add on midgard if available
            },
            memo: swapAction?.metadata.swap?.memo,
            done:
              thorStatus?.stages?.inbound_finalised?.completed &&
              (thorStatus?.stages.swap_finalised?.completed ||
                !thorStatus?.stages.swap_status?.pending),
            error: onlyRefund,
          },
          out: [
            {
              txid: outTxs?.length > 0 ? outTxs[0]?.id : null,
              to: (outTxs?.length > 0 && outTxs[0]?.to_address) || memo?.destAddr,
              asset: outAsset,
              amount: parseInt(outAmount),
              gas:
                outTxs?.length > 0 && outTxs[0]?.gas
                  ? outTxs[0]?.gas?.[0]?.amount
                  : null,
              gasAsset:
                outTxs?.length > 0 && outTxs[0]?.gas
                  ? this.parseMemoAsset(outTxs[0]?.gas?.[0]?.asset, this.pools)
                  : null,
              height: outTxs?.length > 0 ? outTxs[0]?.height : null,
              fees: outboundFees,
              feeAssets: outboundFeeAssets,
              delayBlocksRemaining:
                thorStatus?.stages.outbound_delay?.remaining_delay_blocks || 0,
              outboundDelayRemaining: outboundDelayRemaining || 0,
              outboundETA:
                thorStatus?.stages.outbound_signed?.scheduled_outbound_height -
                this.thorHeight,
              outboundSigned:
                thorStatus?.stages.outbound_signed?.completed ?? undefined,
              done:
                (outTxs?.length > 0 && outTxs[0]?.id) ||
                (!thorStatus?.stages.swap_status?.pending &&
                  (thorStatus?.stages.outbound_signed?.completed ||
                    outAsset?.chain === 'THOR' ||
                    outAsset?.synth ||
                    outAsset?.trade ||
                    outAsset?.secure) &&
                  (thorStatus?.stages.outbound_delay?.completed ?? true)),
            },
            ...(outTxs ?? []).slice(1).map((o) => ({
              txid: o.id,
              to: o.to_address,
              asset: this.parseMemoAsset(o.coins?.[0]?.asset, this.pools),
              amount: parseInt(o.coins?.[0]?.amount ?? 0),
              gas: o.gas ? o.gas?.[0]?.amount : null,
              height: o.height,
              gasAsset: o.gas
                ? this.parseMemoAsset(o.gas?.[0]?.asset, this.pools)
                : null,
              done:
                !!o.id ||
                (!thorStatus?.stages.swap_status?.pending &&
                  (thorStatus?.stages.outbound_signed?.completed ||
                    outAsset?.chain === 'THOR' ||
                    outAsset?.synth ||
                    outAsset?.trade ||
                    outAsset?.secure)),
            })),
          ],
        },
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.tx-header {
  display: flex;
  max-width: 940px;
  gap: 8px;
  align-items: center;
  justify-content: center;
  margin: $space-0 $space-10;

  @include lg {
    margin: auto;
    width: 100%;
  }

  @include md {
    justify-content: flex-start;
  }

  .item {
    border: 1px solid var(--border-color);
    background-color: var(--card-bg-color);
    padding: $space-8;
    border-radius: $radius-s;
    display: flex;
    align-items: center;
    gap: 8px;
    margin: $space-0;
    max-width: 100%;
    min-width: 32px;

    &.tx-id {
      flex: 1 1 auto;
    }

    &:not(.tx-id) {
      flex: 0 0 32px;
      height: 32px;
      width: 32px;
    }

    span {
      color: var(--sec-font-color);
      line-height: 15px;
      height: 16px;
    }

    &:hover {
      span {
        color: var(--primary-color);
      }

      * {
        fill: var(--primary-color);
      }
    }

    * {
      fill: var(--sec-font-color);
    }
  }
}

.contract-overview {
  max-width: 940px;
  margin: $space-12 $space-10 $space-16;
  padding: $space-16;
  border: 1px solid color-mix(in srgb, var(--highlight) 18%, var(--border-color));
  border-radius: $radius-s;
  background:
    linear-gradient(
      180deg,
      color-mix(in srgb, var(--highlight) 8%, var(--card-bg-color)),
      var(--card-bg-color)
    );

  @include lg {
    margin-left: auto;
    margin-right: auto;
    width: 100%;
  }

}

.contract-overview__header {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: $space-12;
  margin-bottom: $space-16;

  @include md {
    flex-direction: row;
    justify-content: space-between;
    align-items: flex-start;
  }
}

.contract-overview__eyebrow {
  display: block;
  margin-bottom: $space-6;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--sec-font-color);
}

.contract-overview__title {
  margin: 0;
  font-size: $font-size-xl;
}

.contract-overview__method {
  display: inline-block;
  margin-top: $space-6;
  color: var(--sec-font-color);
}

.contract-overview__status {
  padding: $space-8 $space-12;
  border-radius: $radius-s;
  font-weight: 600;
  border: 1px solid var(--border-color);
}

.contract-overview__status--success {
  color: var(--green);
  border-color: color-mix(in srgb, var(--green) 45%, var(--border-color));
  background-color: color-mix(in srgb, var(--green) 12%, var(--card-bg-color));
}

.contract-overview__status--error {
  color: var(--red);
  border-color: color-mix(in srgb, var(--red) 45%, var(--border-color));
  background-color: color-mix(in srgb, var(--red) 12%, var(--card-bg-color));
}

.contract-overview__stats {
  display: grid;
  grid-template-columns: 1fr;
  gap: $space-10;
  margin-bottom: $space-16;

  @include sm {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  @include md {
    grid-template-columns: repeat(4, minmax(0, 1fr));
  }
}

.contract-overview__stat,
.contract-overview__panel {
  border: 1px solid var(--border-color);
  border-radius: $radius-s;
  background-color: color-mix(in srgb, var(--card-bg-color) 90%, black);
}

.contract-overview__stat {
  min-height: 88px;
  padding: $space-10 $space-12;
}

.contract-overview__stat-label,
.contract-overview__detail-item small,
.contract-overview__flow-party small {
  display: block;
  margin-bottom: $space-4;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--sec-font-color);
}

.contract-overview__stat-value {
  display: inline-flex;
  align-items: center;
  gap: $space-6;
  font-size: $font-size-md;
  font-weight: 600;
  line-height: 1.25;
}

.contract-overview__stat-icon {
  flex: 0 0 auto;
}

.contract-overview__stat-value--compact {
  font-size: $font-size-sm;
  line-height: 1.3;
}

.contract-overview__stat-value--muted {
  color: var(--sec-font-color) !important;
  opacity: 0.8;
}

.contract-overview__stat-subtext {
  display: block;
  margin-top: $space-4;
  font-size: $font-size-xs;
  color: var(--sec-font-color);
}

.contract-overview__party-link {
  color: inherit;
  text-decoration: none;

  &:visited,
  &:active {
    color: inherit;
  }

  &:hover,
  &:focus-visible {
    color: var(--primary-color);
  }
}

.contract-overview__content {
  display: grid;
  grid-template-columns: 1fr;
  gap: $space-12;

  @include lg {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

.contract-overview__panel {
  padding: $space-12;
}

.contract-overview__accordion-toggle {
  width: 100%;
  display: flex;
  align-items: center;
  gap: $space-8;
  padding: 0;
  border: 0;
  background: transparent;
  color: inherit;
  text-align: left;
  cursor: pointer;
}

.contract-overview__accordion-left {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: $font-size-desktop;
  font-weight: bolder;
  line-height: 1.2;
}

.contract-overview__accordion-icon {
  width: 1rem;
  height: 1rem;
  fill: var(--font-color);
  transition: transform 0.3s ease, fill 0.2s ease;
}

.contract-overview__accordion-icon--open {
  transform: rotate(180deg);
}

.contract-overview__accordion-toggle:hover {
  color: var(--sec-font-color);
}

.contract-overview__accordion-toggle:hover .contract-overview__accordion-icon {
  fill: var(--sec-font-color);
}

.contract-overview__accordion-body {
  font-size: $font-size-sm;
  overflow: hidden;
}

.contract-overview__accordion-body--open {
  margin-top: $space-8;
  padding-top: $space-8;
  border-top: 1px solid var(--border-color);
}

.contract-overview__flow-list,
.contract-overview__detail-list {
  display: grid;
  grid-template-columns: 1fr;
  gap: $space-10;
}

.contract-overview__detail-list {
  @include md {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

.contract-overview__flow-item,
.contract-overview__detail-item {
  border: 1px solid var(--border-color);
  border-radius: $radius-s;
  background-color: var(--bgl-color);
}

.contract-overview__flow-item {
  display: grid;
  grid-template-columns: 1fr;
  gap: $space-8;
  padding: $space-10;

  @include md {
    grid-template-columns: minmax(0, 1fr) auto minmax(0, 1fr);
    gap: $space-12;
    align-items: center;
  }
}

.contract-overview__flow-party {
  min-width: 0;

  div {
    font-size: $font-size-sm;
    font-weight: 500;
    line-height: 1.4;
    word-break: break-word;
  }
}

.contract-overview__flow-item .contract-overview__flow-party:last-child {
  @include md {
    text-align: right;
  }
}

.contract-overview__flow-center {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: $space-6 $space-10;
  border-radius: $radius-s;
  background-color: color-mix(in srgb, var(--highlight) 10%, var(--bgl-color));
}

.contract-overview__flow-amount {
  font-size: $font-size-sm;
  font-weight: 700;
  line-height: 1.2;
}

.contract-overview__flow-token {
  display: inline-flex;
  align-items: center;
  gap: $space-4;

  small {
    margin: 0;
    font-size: $font-size-xs;
    color: var(--sec-font-color);
  }
}

.contract-overview__detail-item {
  padding: $space-10;

  div {
    font-size: $font-size-sm;
    font-weight: 500;
    line-height: 1.4;
    word-break: break-word;
  }
}

.contract-overview__detail-item--full {
  @include md {
    grid-column: 1 / -1;
  }
}

.contract-overview__detail-item--multiline {
  div {
    white-space: pre-line;
  }
}

.qr-icon {
  fill: var(--font-color);
  width: 16px;
  height: 16px;
}

.tx-wrapper {
  position: relative;

  .arrow {
    display: none;
    flex: 1;
    justify-content: center;
    align-items: center;

    .icon {
      margin-right: $space-0;
    }

    @include md {
      display: flex;
    }
  }
}
.tx-container {
  border: 1px solid var(--border-color);
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  background: var(--card-bg-color);
  border-radius: $radius-s;
  padding: $space-20;
  gap: $space-10;
}

.tx-contain {
  display: flex;
  flex-direction: column;
  gap: $space-10;

  .asset-icon-container {
    margin-top: $space-10;
    display: flex;
    align-items: center;

    span {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      max-width: 300px;
    }
  }

  .address {
    margin-top: $space-10;
  }

  .txid {
    width: 300px;
    display: flex;
    align-items: center;
    gap: 10px;

    .tx-hash {
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
  }
}

.icon {
  fill: var(--sec-font-color);
  height: 1.5rem;

  &.small {
    margin-right: $space-0;
    height: 0.8rem;
    width: 0.8rem;
  }
}

.extra-details {
  margin-top: $space-16;

  .pool-box {
    margin: $space-5 $space-0;
    display: flex;
    align-items: center;
  }
}

.utility,
.tx-date {
  padding: $space-0 $space-16;
}

.utility {
  justify-content: space-between;
  gap: $space-16;
}

.asset-text {
  font-size: $font-size-md;
}

.tx-id {
  flex-shrink: 5;
  span {
    overflow: hidden;
    text-overflow: ellipsis;
  }
}
</style>
