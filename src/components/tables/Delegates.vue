<template>
  <Loader :data="delegates">
    <TableWrapper
      v-bind="$attrs"
      :columns="columns"
      :rows="delegates"
      :no-data-message="$t('COMMON.NO_RESULTS')"
      @on-sort-change="emitSortChange"
    >
      <template slot-scope="data">
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

        <div v-else-if="data.column.field === 'forgingStatus'" class="text-0">
          <button v-tooltip="statusTooltip(data.row)" role="img" :aria-label="tooltipContent(data.row)">
            <img class="mx-auto" width="19px" :src="require(`@/assets/images/icons/${status(data.row)}.svg`)" />
          </button>
        </div>

        <div v-else-if="data.column.field === 'votes'">
          <span v-tooltip="$t('COMMON.SUPPLY_PERCENTAGE')" class="text-grey text-2xs mr-1">
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

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { IDelegate, ISortParameters } from "@/interfaces";

@Component
export default class TableDelegates extends Vue {
  @Prop({
    required: true,
    validator: value => {
      return Array.isArray(value) || value === null;
    },
  })
  public delegates: IDelegate[] | null;
  @Prop({ required: false, default: false }) public showStandby: boolean;

  get columns() {
    const columns = [
      {
        label: this.$t("COMMON.RANK"),
        field: "rank",
        type: "number",
        thClass: "start-cell w-32",
        tdClass: "start-cell w-32",
      },
      {
        label: this.$t("WALLET.DELEGATE.USERNAME"),
        field: "username",
        thClass: `${this.showStandby ? "end-cell sm:base-cell text-left" : ""}`,
        tdClass: `${this.showStandby ? "end-cell sm:base-cell text-left" : ""}`,
      },
      {
        label: this.$t("PAGES.DELEGATE_MONITOR.FORGED_BLOCKS"),
        field: "blocks.produced",
        type: "number",
        thClass: "text-left hidden xl:table-cell",
        tdClass: "text-left hidden xl:table-cell",
      },
      {
        label: this.$t("PAGES.DELEGATE_MONITOR.LAST_FORGED"),
        field: "lastBlockHeight",
        type: "number",
        sortFn: this.sortByLastBlockHeight,
        thClass: "text-left hidden sm:table-cell",
        tdClass: "text-left hidden sm:table-cell",
      },
      {
        label: this.$t("PAGES.DELEGATE_MONITOR.STATUS.TITLE"),
        field: "forgingStatus",
        type: "number",
        thClass: "end-cell md:base-cell text-center",
        tdClass: "end-cell md:base-cell text-center",
      },
      {
        label: this.$t("PAGES.DELEGATE_MONITOR.VOTES"),
        field: "votes",
        type: "number",
        thClass: `end-cell hidden ${this.showStandby ? "sm" : "md"}:table-cell`,
        tdClass: `end-cell hidden ${this.showStandby ? "sm" : "md"}:table-cell`,
      },
    ];

    if (this.showStandby) {
      // remove the columns for blocks, last forged and status
      const index = columns.findIndex(el => {
        return el.field === "blocks.produced";
      });
      columns.splice(index, 3);
    }

    return columns;
  }

  private lastForgingTime(delegate: IDelegate) {
    return delegate.blocks.last
      ? // Comment to keepe ts-ignore in check
        // @ts-ignore
        this.readableTimestampAgo(delegate.blocks.last.timestamp.unix)
      : this.$i18n.t("PAGES.DELEGATE_MONITOR.NEVER");
  }

  private statusTooltip(row: any) {
    return {
      trigger: "hover click",
      content: this.tooltipContent(row),
      classes: [`tooltip-bg-${this.status(row)}`, "font-sans"],
    };
  }

  private tooltipContent(row: any) {
    // @ts-ignore
    const status = {
      0: this.$i18n.t("PAGES.DELEGATE_MONITOR.STATUS.FORGING"),
      1: this.$i18n.t("PAGES.DELEGATE_MONITOR.STATUS.MISSING"),
      2: this.$i18n.t("PAGES.DELEGATE_MONITOR.STATUS.NOT_FORGING"),
      3: this.$i18n.t("PAGES.DELEGATE_MONITOR.STATUS.NEVER_FORGED"),
    }[row.forgingStatus];

    const lastBlock = row.blocks.last;

    return lastBlock
      ? `[${status}] ${this.$i18n.t("PAGES.DELEGATE_MONITOR.TOOLTIP", {
          height: lastBlock.height,
        })} ${
          // @ts-ignore
          this.readableTimestamp(lastBlock.timestamp.unix)
        }`
      : status;
  }

  private status(row: any) {
    // @ts-ignore
    return {
      0: "forging",
      1: "missed-block",
      2: "not-forging",
      3: "not-forging", // never-forged
    }[row.forgingStatus];
  }

  private sortByLastBlockHeight(x: number, y: number, col: number, rowX: any, rowY: any) {
    const heightX = rowX.blocks.last ? rowX.blocks.last.height : -1;
    const heightY = rowY.blocks.last ? rowY.blocks.last.height : -1;

    return heightX > heightY ? -1 : heightX < heightY ? 1 : 0;
  }

  private emitSortChange(params: ISortParameters[]) {
    this.$emit("on-sort-change", params[0]);
  }
}
</script>
