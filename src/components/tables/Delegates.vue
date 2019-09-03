<template>
  <Loader :data="delegates">
    <TableWrapper
      v-bind="$attrs"
      :columns="columns"
      :rows="delegates"
      :no-data-message="$t('COMMON.NO_RESULTS')"
      @on-sort-change="emitSortChange"
    >
      <template
        slot-scope="data"
      >
        <div v-if="data.column.field === 'username'">
          <LinkWallet :address="data.row.address">
            {{ data.row.username }}
          </LinkWallet>
        </div>

        <div v-else-if="data.column.field === 'blocks.produced'">
          {{ readableNumber(data.row.blocks.produced, 0) }}
        </div>

        <div v-else-if="data.column.field === 'lastBlockHeight'">
          {{ lastForgingTime(data.row) }}
        </div>

        <div v-else-if="data.column.field === 'forgingStatus'">
          <img
            v-tooltip="statusMessage(data.row)"
            class="mx-auto"
            width="19px"
            :src="require(`@/assets/images/icons/${statusImage(data.row)}.svg`)"
          >
        </div>

        <div v-else-if="data.column.field === 'votes'">
          <span
            v-tooltip="$t('COMMON.SUPPLY_PERCENTAGE')"
            class="text-grey text-2xs mr-1"
          >
            {{ percentageString(data.row.production.approval) }}
          </span>
          {{ readableCrypto(data.row.votes, true, 2) }}
        </div>

        <span v-else>
          {{ data.formattedRow[data.column.field] }}
        </span>
      </template>
    </TableWrapper>
  </Loader>
</template>

<script type="text/ecmascript-6">
export default {
  name: 'TableDelegates',

  props: {
    delegates: {
      validator: value => {
        return Array.isArray(value) || value === null
      },
      required: true
    },
    showStandby: {
      type: Boolean,
      required: false,
      default: false
    }
  },

  computed: {
    columns () {
      const columns = [
        {
          label: this.$t('COMMON.RANK'),
          field: 'rank',
          type: 'number',
          thClass: 'start-cell w-32',
          tdClass: 'start-cell w-32'
        },
        {
          label: this.$t('WALLET.DELEGATE.USERNAME'),
          field: 'username',
          thClass: `${this.showStandby ? 'end-cell sm:base-cell text-left' : ''}`,
          tdClass: `${this.showStandby ? 'end-cell sm:base-cell text-left' : ''}`
        },
        {
          label: this.$t('PAGES.DELEGATE_MONITOR.FORGED_BLOCKS'),
          field: 'blocks.produced',
          type: 'number',
          thClass: 'text-left hidden xl:table-cell',
          tdClass: 'text-left hidden xl:table-cell'
        },
        {
          label: this.$t('PAGES.DELEGATE_MONITOR.LAST_FORGED'),
          field: 'lastBlockHeight',
          type: 'number',
          sortFn: this.sortByLastBlockHeight,
          thClass: 'text-left hidden sm:table-cell',
          tdClass: 'text-left hidden sm:table-cell'
        },
        {
          label: this.$t('PAGES.DELEGATE_MONITOR.STATUS.TITLE'),
          field: 'forgingStatus',
          type: 'number',
          thClass: 'end-cell md:base-cell text-center',
          tdClass: 'end-cell md:base-cell text-center'
        },
        {
          label: this.$t('PAGES.DELEGATE_MONITOR.VOTES'),
          field: 'votes',
          type: 'number',
          thClass: `end-cell hidden ${this.showStandby ? 'sm' : 'md'}:table-cell`,
          tdClass: `end-cell hidden ${this.showStandby ? 'sm' : 'md'}:table-cell`
        }
      ]

      if (this.showStandby) {
        // remove the columns for blocks, last forged and status
        const index = columns.findIndex(el => {
          return el.field === 'blocks.produced'
        })
        columns.splice(index, 3)
      }

      return columns
    }
  },

  methods: {
    lastForgingTime (delegate) {
      return delegate.blocks.last ? this.readableTimestampAgo(delegate.blocks.last.timestamp.unix) : this.$i18n.t('PAGES.DELEGATE_MONITOR.NEVER')
    },

    statusMessage (row) {
      const status = {
        0: this.$i18n.t('PAGES.DELEGATE_MONITOR.STATUS.FORGING'),
        1: this.$i18n.t('PAGES.DELEGATE_MONITOR.STATUS.MISSING'),
        2: this.$i18n.t('PAGES.DELEGATE_MONITOR.STATUS.NOT_FORGING'),
        3: this.$i18n.t('PAGES.DELEGATE_MONITOR.STATUS.NEVER_FORGED')
      }[row.forgingStatus]

      const lastBlock = row.blocks.last

      return {
        trigger: 'hover click',
        content: lastBlock ? `[${status}] ${
          this.$i18n.t('PAGES.DELEGATE_MONITOR.TOOLTIP', { height: lastBlock.height })
        } ${this.readableTimestamp(lastBlock.timestamp.unix)}`
          : status,
        classes: [`tooltip-bg-${row.forgingStatus}`, 'font-sans']
      }
    },

    statusImage (row) {
      return {
        0: 'forging',
        1: 'missed-block',
        2: 'not-forging',
        3: 'not-forging'
      }[row.forgingStatus]
    },

    statusColor (row) {
      return {
        0: '#46b02e', // Forging
        1: '#f6993f', // Missing
        2: '#ef192d', // Not forging
        3: '#ef192d' // Never forged
      }[row.forgingStatus]
    },

    sortByLastBlockHeight (x, y, col, rowX, rowY) {
      const heightX = rowX.blocks.last ? rowX.blocks.last.height : -1
      const heightY = rowY.blocks.last ? rowY.blocks.last.height : -1

      return heightX > heightY ? -1 : (heightX < heightY ? 1 : 0)
    },

    emitSortChange (params) {
      this.$emit('on-sort-change', params[0])
    }
  }
}
</script>