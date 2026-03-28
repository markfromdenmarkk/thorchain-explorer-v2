<template>
  <div>
    <card v-if="state && explorers.length == 0">
      <div class="balance-container">
        <div class="balance-content-wrapper">
          <div class="balance-side-column">
            <div class="balance-info">
              <span class="title-balance">Balances</span>
              <div class="balance-label">
                <span>Wallet Balance</span>
                <skeleton-item :loading="loading" class="balance-content">
                  <span
                    v-if="walletBalanceUSD > 0"
                    v-tooltip="formatCurrency(walletBalanceUSD)"
                    class="mono"
                  >
                    {{ walletBalanceUSD | currency }}
                  </span>
                  <span v-else>-</span>
                </skeleton-item>
              </div>

              <div v-if="isNodeAddress" class="balance-label">
                <span>{{ 'Node Balance' }}</span>
                <skeleton-item
                  :loading="loading || !nodes"
                  class="balance-content"
                >
                  <asset-icon
                    :asset="{ ticker: 'RUNE', chain: 'THOR' }"
                    :height="'16px'"
                    :chain="false"
                  />
                  <div class="bonds">
                    <span
                      v-if="totalBond !== undefined"
                      v-tooltip="formatCurrency((runePrice * totalBond) / 1e8)"
                      class="mono"
                    >
                      {{ balanceFormat(totalBond / 1e8) }} RUNE
                      <nuxt-link
                        v-if="isNodeAddress"
                        :to="'/node/' + address"
                        class="clickable"
                        style="margin-left: 0.5rem"
                      >
                        View Node
                      </nuxt-link>
                    </span>
                    <span v-else class="mono">-</span>
                  </div>
                </skeleton-item>
              </div>

              <div class="balance-label">
                <span>Bond Balance</span>
                <skeleton-item
                  :loading="loading || !nodes"
                  class="balance-content"
                >
                  <asset-icon
                    :asset="{ ticker: 'RUNE', chain: 'THOR' }"
                    :height="'16px'"
                    :chain="false"
                  />
                  <div class="bonds">
                    <span
                      v-if="bonds && bonds.total !== undefined"
                      v-tooltip="formatCurrency(runePrice * bonds.total)"
                      class="mono"
                    >
                      {{ balanceFormat(bonds.total) }} RUNE
                    </span>
                    <span v-else class="mono">-</span>
                  </div>
                </skeleton-item>
              </div>

              <div class="balance-label">
                <span>Total Value</span>
                <skeleton-item :loading="loading" class="balance-content">
                  <span v-if="portfolioTotalUSD > 0" class="mono">
                    {{ portfolioTotalUSD | currency }}
                  </span>
                  <span v-else>-</span>
                </skeleton-item>
              </div>
            </div>
          </div>

          <div
            v-if="networkExposureRows.length > 0"
            class="network-balances-section"
          >
            <span class="title-balance">Assets by Network</span>
            <div class="network-balances-list">
              <div
                v-for="network in networkExposureRows"
                :key="network.chain"
                class="network-balance-row"
              >
                <div class="network-summary">
                  <asset-icon
                    :asset="baseChainAsset(network.chain)"
                    :chain="false"
                  />
                  <div class="network-meta">
                    <div class="network-name">
                      {{ network.name }}
                    </div>
                    <div class="network-assets">
                      {{ network.assetCount }} asset{{
                        network.assetCount === 1 ? '' : 's'
                      }}
                    </div>
                  </div>
                </div>
                <div class="token-value">
                  <span v-if="network.value > 0">
                    ${{ network.value | number('0,0.00') }}
                  </span>
                  <span v-else>-</span>
                </div>
              </div>
            </div>
          </div>

          <div v-if="sortedAssetTokens.length > 0" class="other-balances-section">
            <span class="title-balance">Assets by Value</span>
            <div class="other-balances-list">
              <div
                v-for="token in sortedAssetTokens"
                :key="`${token.asset.chain}-${token.asset.ticker}-${token.asset.symbol || ''}`"
                class="other-balance-row"
              >
                <div class="token-summary">
                  <asset-icon :asset="token.asset" :chain="false" />
                  <div class="token-meta">
                    <div class="token-name">
                      {{ showAsset(token.asset) }}
                    </div>
                    <div class="token-quantity">
                      {{ token.quantity }} {{ token.asset.ticker }}
                    </div>
                  </div>
                </div>
                <div class="token-value">
                  <span v-if="token.price > 0 && !isNaN(token.price)">
                    ${{ token.value | number('0,0.00') }}
                  </span>
                  <span v-else>-</span>
                </div>
              </div>
            </div>
          </div>

          <div
            v-if="balanceAllocationData.length > 0"
            class="balance-allocation-column"
          >
            <span class="title-balance">Assets by Allocation</span>
            <div class="balance-chart-section">
              <pie-chart
                :pie-data="balanceAllocationData"
                :extra-series="chartExtraSeries"
                :extra="chartExtra"
                :height="'320px'"
              />
            </div>
            <div class="allocation-legend">
              <div
                v-for="item in balanceAllocationData"
                :key="item.name"
                class="allocation-legend-item"
              >
                <span
                  class="allocation-legend-dot"
                  :style="{ backgroundColor: item.itemStyle.color }"
                ></span>
                <span class="allocation-legend-label">{{ item.name }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </card>
    <card v-else title="Chain Explorers" class="explorers-card">
      <div class="explorers">
        <div v-for="explorer in explorers" :key="explorer.chain">
          <a
            class="explorer-link mini-bubble info hoverable"
            :href="explorer.url"
            target="_blank"
            rel="noopener noreferrer"
          >
            <asset-icon
              :height="'1.2rem'"
              :asset="baseChainAsset(explorer.chain)"
            ></asset-icon>
            {{ explorer.chain }}
            <external-icon class="ext-icon" />
          </a>
        </div>
      </div>
    </card>
  </div>
</template>

<script>
import { bnOrZero } from '@xchainjs/xchain-util'
import { mapGetters } from 'vuex'
import { orderBy } from 'lodash'
import { validate } from '@swyftx/api-crypto-address-validator'
import { assetFromString, getExplorerAddressUrl } from '~/utils'
import ExternalIcon from '@/assets/images/external.svg?inline'
import PieChart from '~/components/PieChart.vue'

export default {
  components: {
    ExternalIcon,
    PieChart,
  },
  props: ['state', 'loading', 'address'],
  data() {
    return {
    }
  },
  computed: {
    ...mapGetters({
      runePrice: 'getRunePrice',
      pools: 'getPools',
      nodes: 'getNodesData',
    }),
    tokenRows() {
      if (!this.state) {
        return []
      }

      const ret = []
      for (let i = 0; i < this.state.length; i++) {
        const e = this.state[i]
        let poolAsset
        this.pools?.forEach((p) => {
          const pa = assetFromString(p.asset)
          if (pa.chain === e.asset?.chain && pa?.ticker === e.asset?.ticker) {
            poolAsset = p
          }
        })

        if (e.asset?.ticker === 'RUNE' && e.asset?.chain === 'THOR') {
          poolAsset = {
            assetPriceUSD: this.runePrice,
          }
        }

        ret.push({
          asset: e.asset,
          quantity: e.quantity,
          price: bnOrZero(poolAsset?.assetPriceUSD).toFixed(2),
          value: bnOrZero(poolAsset?.assetPriceUSD * e.quantity).toFixed(2),
          type: this.getAssetType(e.asset),
        })
      }

      return ret
    },
    runeToken() {
      return this.tokenRows.find(
        (token) =>
          token?.asset?.ticker === 'RUNE' && token?.asset?.chain === 'THOR'
      )
    },
    walletBalanceUSD() {
      return this.tokenRows.reduce((sum, token) => sum + Number(token.value), 0)
    },
    otherTokens() {
      return this.tokenRows.filter(
        (token) =>
          !(token.asset?.ticker === 'RUNE' && token.asset?.chain === 'THOR')
      )
    },
    sortedAssetTokens() {
      return orderBy(
        this.tokenRows.filter((token) => token.asset),
        [(token) => Number(token.value), (token) => token.asset?.ticker],
        ['desc', 'asc']
      )
    },
    networkExposureRows() {
      const networks = this.tokenRows.reduce((acc, token) => {
        const chain = token?.asset?.chain
        if (!chain) {
          return acc
        }

        if (!acc[chain]) {
          acc[chain] = {
            chain,
            name: this.getNetworkName(chain),
            value: 0,
            assetCount: 0,
          }
        }

        acc[chain].value += Number(token.value || 0)
        acc[chain].assetCount += 1
        return acc
      }, {})

      return orderBy(
        Object.values(networks),
        [(network) => Number(network.value), 'name'],
        ['desc', 'asc']
      )
    },
    isNodeAddress() {
      return this.nodes?.some((node) => node.node_address === this.address)
    },
    totalBond() {
      const foundNode = this.nodes?.find(
        (node) => node.node_address === this.address
      )
      if (foundNode) {
        return foundNode.total_bond
      }
      return undefined
    },
    bonds() {
      if (!this.nodes) {
        return undefined
      }
      const ret = { total: 0 }

      for (let i = 0; i < this.nodes.length; i++) {
        const node = this.nodes[i]
        const bond = node.bond_providers?.providers?.find(
          (n) => n.bond_address === this.address
        )
        if (bond !== undefined) {
          ret.total += +bond.bond / 1e8
        }
      }
      return ret
    },
    bondBalanceRune() {
      if (this.totalBond !== undefined) {
        return this.totalBond / 1e8
      }
      if (this.bonds?.total > 0) {
        return this.bonds.total
      }
      return 0
    },
    bondBalanceUSD() {
      return this.bondBalanceRune * Number(this.runePrice || 0)
    },
    portfolioTotalUSD() {
      return this.walletBalanceUSD + this.bondBalanceUSD
    },
    totalBalance() {
      let ret = this.runeToken.quantity
      if (this.bonds?.total > 0) {
        ret += this.bonds.total
      }
      return ret
    },
    balanceAllocationData() {
      const palette = [
        '#19c2ee',
        '#5af0d1',
        '#8a7dff',
        '#f5d86c',
        '#ff9f6e',
        '#7dd3fc',
        '#a78bfa',
        '#4ade80',
      ]

      const data = this.tokenRows
        .filter((token) => Number(token.value) > 0)
        .map((token) => ({
          name: this.showAsset(token.asset),
          value: Number(token.value),
        }))

      if (this.bondBalanceUSD > 0) {
        data.push({
          name: 'Bonded RUNE',
          value: this.bondBalanceUSD,
        })
      }

      return data.map((item, index) => ({
        ...item,
        itemStyle: {
          color: palette[index % palette.length],
        },
      }))
    },
    chartExtraSeries() {
      return {
        center: ['50%', '50%'],
        radius: ['54%', '82%'],
        label: {
          show: false,
        },
      }
    },
    chartExtra() {
      return {
        legend: {
          show: false,
        },
        tooltip: {
          trigger: 'item',
          confine: true,
          formatter: (a) => {
            return `${a.name}: <small>${this.formatCurrency(
              a?.data?.value
            )}</small> <span class='mono'>(${a.percent}%)</span>`
          },
        },
      }
    },
    explorers() {
      const blockChains = [
        'btc',
        'eth',
        'doge',
        'bch',
        'ltc',
        'atom',
        'xrp',
        'trx',
      ]

      const explorers = []
      for (let i = 0; i < blockChains.length; i++) {
        let chain = blockChains[i]
        const res = validate(this.address, chain)

        if (res && chain === 'eth') {
          const evms = ['ETH', 'BSC', 'AVAX', 'BASE']
          evms.forEach((e) => {
            explorers.push({
              chain: e,
              url: getExplorerAddressUrl(e, this.address),
            })
          })
        } else if (res) {
          // Change atom to GAIA. its chain
          if (chain === 'atom') {
            chain = 'gaia'
          }
          if (chain === 'trx') {
            chain = 'tron'
          }
          explorers.push({
            chain: chain.toUpperCase(),
            url: getExplorerAddressUrl(chain.toUpperCase(), this.address),
          })
        }
      }

      return explorers
    },
  },
  methods: {
    getAssetType(asset) {
      if (asset?.synth) {
        return 'Synth'
      } else if (asset?.trade) {
        return 'Trade'
      } else {
        return 'Native'
      }
    },
    getNetworkName(chain) {
      const names = {
        ARB: 'Arbitrum',
        AVAX: 'Avalanche',
        BASE: 'Base',
        BCH: 'Bitcoin Cash',
        BSC: 'Binance Smart Chain (BSC)',
        BTC: 'Bitcoin',
        DOGE: 'Dogecoin',
        ETH: 'Ethereum',
        GAIA: 'Cosmos',
        LTC: 'Litecoin',
        THOR: 'THORChain',
        TRON: 'Tron',
        XRP: 'XRP Ledger',
      }

      return names[chain] || chain
    },
  },
}
</script>

<style lang="scss" scoped>
.bonds {
  width: 100%;
}

.balance-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 240px;
  gap: 24px;
  flex: 1;
}
.balance-content-wrapper {
  display: flex;
  gap: 24px;
  align-items: flex-start;
  justify-content: flex-start;
  flex-wrap: wrap;

  @include md {
    display: grid;
    grid-template-columns: minmax(132px, 0.62fr) minmax(210px, 1fr) minmax(280px, 1.15fr) minmax(360px, 1.35fr);
    column-gap: 20px;
    row-gap: 24px;
    align-items: start;
  }
}

.balance-side-column {
  flex: 1 1 100%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 24px;
  min-width: 0;

  @include md {
    padding-left: 24px;
  }
}

.balance-info {
  display: flex;
  flex-direction: column;
  gap: 16px;
  min-width: 0;
  width: 100%;

  .balance-label {
    letter-spacing: 1px;
    display: flex;
    flex-direction: column;
    gap: 5px;
    font-size: $font-size-xs;
  }
  .balance-value {
    display: flex;
    align-items: center;
    font-size: $font-size-desktop;
    font-weight: bold;
    align-items: center;
  }
  .balance-content {
    display: flex;
  }
}

.title-balance {
  font-weight: bold;
}

.other-balances-section {
  flex: 1 1 100%;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: $space-12;

  @include md {
    padding-left: 12px;
  }
}

.network-balances-section {
  flex: 1 1 100%;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: $space-12;
}

.balance-allocation-column {
  flex: 1 1 100%;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: $space-12;
}

.balance-side-column,
.network-balances-section,
.other-balances-section,
.balance-allocation-column {
  padding-top: 2px;
}

.other-balances-list {
  display: flex;
  flex-direction: column;
  gap: $space-10;
}

.network-balances-list {
  display: flex;
  flex-direction: column;
  gap: $space-10;
}

.other-balance-row {
  display: flex;
  justify-content: space-between;
  gap: $space-12;
  align-items: center;
  padding-bottom: $space-10;
  border-bottom: 1px solid var(--border-color);

  &:last-child {
    border-bottom: none;
    padding-bottom: 0;
  }

  .token-summary {
    display: flex;
    align-items: center;
    gap: $space-10;
    min-width: 0;
  }

  .token-meta {
    display: flex;
    flex-direction: column;
    gap: 4px;
    min-width: 0;
  }

  .token-name {
    font-size: $font-size-sm;
    color: var(--font-color);
    line-height: 1.2;
    word-break: break-word;
  }

  .token-quantity {
    font-size: $font-size-xs;
    color: var(--sec-font-color);
    line-height: 1.2;
    word-break: break-word;
  }

  .token-value {
    font-size: $font-size-sm;
    color: var(--font-color);
    white-space: nowrap;
    text-align: right;
  }
}

.network-balance-row {
  display: flex;
  justify-content: space-between;
  gap: $space-12;
  align-items: flex-start;
  padding-bottom: $space-10;
  border-bottom: 1px solid var(--border-color);

  &:last-child {
    border-bottom: none;
    padding-bottom: 0;
  }

  .token-value {
    font-size: $font-size-sm;
    color: var(--font-color);
    white-space: nowrap;
    text-align: right;
  }
}

.network-summary {
  display: flex;
  align-items: center;
  gap: $space-10;
  min-width: 0;
}

.network-meta {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 0;
}

.network-name {
  font-size: $font-size-sm;
  color: var(--font-color);
  line-height: 1.2;
  word-break: break-word;
}

.network-assets {
  font-size: $font-size-xs;
  color: var(--sec-font-color);
  line-height: 1.2;
}

.balance-chart-section {
  border-radius: $radius-md;
  width: 360px;
  max-width: 360px;
  margin-left: auto;
}

.allocation-legend {
  display: flex;
  flex-wrap: wrap;
  gap: $space-10 $space-16;
  width: 360px;
  margin-left: auto;
}

.allocation-legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
}

.allocation-legend-dot {
  width: 12px;
  height: 12px;
  border-radius: 999px;
  flex-shrink: 0;
}

.allocation-legend-label {
  font-size: $font-size-xs;
  color: var(--font-color);
  line-height: 1.2;
}

.mono {
  font-size: $font-size-sm !important;
  color: var(--sec-font-color);
  display: flex;
  justify-content: space-between;
  width: 100%;
  align-items: center;
}
.thor-rune-details {
  padding: $space-12;
  margin-bottom: $space-24;
}

.token-details {
  margin-top: $space-10;
  padding: $space-10;
  background-color: var(--bg-color-light);
  border-radius: $radius-md;
}

.detail-item {
  font-size: $font-size-desktop;
  padding: $space-10;
}

button[disabled] {
  cursor: not-allowed;
  opacity: 0.6;
}

.title-explorers {
  font-weight: bold;
  border-bottom: 1px solid var(--border-color);
  margin: $space-0;
  padding-bottom: $space-14;
  margin-bottom: $space-10;
}

.explorers-card {
  max-width: 500px;
  min-height: 100%;
}

.explorers {
  display: flex;
  flex-wrap: wrap;
  gap: $space-8;

  .explorer-link {
    display: flex;
    align-items: center;
    height: 2rem;

    .ext-icon {
      fill: currentColor;
      height: 0.8rem;
    }
  }
}
</style>
