<template>
  <dialog-base :visible.sync="isVisible">
    <template #title>
      <slot name="title">
        <span class="el-dialog__title">{{ t('confirmBridgeTransactionDialog.confirmTransaction') }}</span>
      </slot>
    </template>
    <slot name="content-title" />
    <div :class="assetsClasses">
      <div class="tokens-info-container">
        <span class="token-value">{{ formattedAmount }}</span>
        <div v-if="asset" class="token">
          <i class="s-icon--network s-icon-sora" />
          {{ tokenSymbol }}
        </div>
      </div>
      <s-icon class="icon-divider" name="arrows-arrow-bottom-24" />
      <div class="tokens-info-container">
        <span class="token-value">{{ formattedAmount }}</span>
        <div v-if="asset" class="token token-ethereum">
          <i :class="`s-icon--network s-icon-${getEvmIcon(evmNetwork)}`" />
          {{ tokenSymbol }}
        </div>
      </div>
    </div>
    <s-divider class="s-divider--dialog" />
    <bridge-transaction-details
      :evm-token-symbol="evmTokenSymbol"
      :evm-network-fee="evmNetworkFee"
      :sora-network-fee="soraNetworkFee"
    />
    <template #footer>
      <s-button
        type="primary"
        class="s-typography-button--large"
        :loading="loading"
        :disabled="isConfirmButtonDisabled"
        @click="handleConfirm"
      >
        <template v-if="!isValidNetworkType">
          {{ t('confirmBridgeTransactionDialog.changeNetwork') }}
        </template>
        <template v-else-if="isInsufficientBalance">
          {{ t('confirmBridgeTransactionDialog.insufficientBalance', { tokenSymbol }) }}
        </template>
        <template v-else>
          {{ confirmText }}
        </template>
      </s-button>
    </template>
  </dialog-base>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator';
import { components, mixins } from '@soramitsu/soraneo-wallet-web';
import { CodecString, BridgeNetworks } from '@sora-substrate/util';
import type { Asset } from '@sora-substrate/util/build/assets/types';

import TranslationMixin from '@/components/mixins/TranslationMixin';
import NetworkFormatterMixin from '@/components/mixins/NetworkFormatterMixin';
import { EvmSymbol, ZeroStringValue, Components } from '@/consts';
import { lazyComponent } from '@/router';

@Component({
  components: {
    DialogBase: components.DialogBase,
    BridgeTransactionDetails: lazyComponent(Components.BridgeTransactionDetails),
  },
})
export default class ConfirmBridgeTransactionDialog extends Mixins(
  mixins.FormattedAmountMixin,
  mixins.LoadingMixin,
  mixins.DialogMixin,
  TranslationMixin,
  NetworkFormatterMixin
) {
  @Prop({ default: ZeroStringValue, type: String }) readonly amount!: string;
  @Prop({ default: () => undefined, type: Object }) readonly asset!: Nullable<Asset>;
  @Prop({ default: BridgeNetworks.ETH_NETWORK_ID, type: Number }) readonly evmNetwork!: BridgeNetworks;
  @Prop({ default: EvmSymbol.ETH, type: String }) readonly evmTokenSymbol!: string;
  @Prop({ default: ZeroStringValue, type: String }) readonly evmNetworkFee!: CodecString;
  @Prop({ default: ZeroStringValue, type: String }) readonly soraNetworkFee!: CodecString;
  @Prop({ default: true, type: Boolean }) readonly isValidNetworkType!: boolean;
  @Prop({ default: true, type: Boolean }) readonly isSoraToEvm!: boolean;
  @Prop({ default: false, type: Boolean }) readonly isInsufficientBalance!: boolean;
  @Prop({ default: '', type: String }) readonly confirmButtonText!: string;

  get confirmText(): string {
    return this.confirmButtonText || this.t('confirmBridgeTransactionDialog.buttonConfirm');
  }

  get isConfirmButtonDisabled(): boolean {
    return !this.isValidNetworkType || this.isInsufficientBalance;
  }

  get assetsClasses(): Array<string> {
    const assetsClass = 'tokens';
    const classes = [assetsClass];

    if (!this.isSoraToEvm) {
      classes.push(`${assetsClass}--reverse`);
    }

    return classes;
  }

  get formattedAmount(): string {
    return this.amount ? this.formatStringValue(this.amount, this.asset?.decimals) : '';
  }

  get formattedSoraNetworkFee(): string {
    return this.formatCodecNumber(this.soraNetworkFee);
  }

  get formattedEvmNetworkFee(): string {
    return this.formatCodecNumber(this.evmNetworkFee);
  }

  get tokenSymbol(): string {
    return this.asset?.symbol || '';
  }

  async handleConfirm(): Promise<void> {
    this.$emit('confirm', true);
    this.closeDialog();
  }
}
</script>

<style lang="scss" scoped>
.tokens {
  display: flex;
  flex-direction: column;
  font-size: var(--s-heading2-font-size);
  line-height: var(--s-line-height-small);
  &-info-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-weight: 800;
  }
  &--reverse {
    flex-direction: column-reverse;
  }
}
@include vertical-divider;
@include vertical-divider('s-divider--dialog', $inner-spacing-medium);
.token {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  white-space: nowrap;
  letter-spacing: var(--s-letter-spacing-mini);
  &-value {
    margin-right: $inner-spacing-medium;
  }
  &-logo {
    display: block;
    margin-right: $inner-spacing-medium;
    flex-shrink: 0;
  }
  .s-icon {
    &-sora,
    &-eth {
      margin-right: $inner-spacing-medium;
      font-size: 21px;
    }
  }
}
</style>
