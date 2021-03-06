<template>
    <div class="unstash-confirmation">
        <tx-confirmation
            :tx="tx"
            confirmation-comp-name="unstash-confirmation"
            send-button-label="Unstash"
            password-label="Please enter your wallet password to unstash your rewards"
            :on-send-transaction-success="onSendTransactionSuccess"
            @change-component="onChangeComponent"
        >
            <h2 class="cont-with-back-btn">
                <span>Unstash Rewards</span>
                <button type="button" class="btn light" @click="onBackBtnClick">Back</button>
            </h2>

            <div class="transaction-info">
                <div class="row no-collapse">
                    <div class="col-3 f-row-label">From</div>
                    <div class="col break-word">
                        {{ currentAccount.address }}
                        <span class="f-row-label">( {{ toFTM(currentAccount.balance) }} FTM )</span>
                    </div>
                </div>

                <div class="row no-collapse">
                    <div class="col-3 f-row-label">Amount</div>
                    <div class="col break-word">{{ toFTM(accountInfo.stashed) }} FTM</div>
                </div>
            </div>

            <template #window-content>
                <ledger-confirmation-content :to="tx.to" :amount="0" />
            </template>
        </tx-confirmation>
    </div>
</template>

<script>
import TxConfirmation from '../TxConfirmation/TxConfirmation.vue';
import { mapGetters } from 'vuex';
import sfcUtils from 'fantom-ledgerjs/src/sfc-utils.js';
import { toFTM } from '../../utils/transactions.js';
import LedgerConfirmationContent from '../LedgerConfirmationContent/LedgerConfirmationContent.vue';

export default {
    name: 'UnstashConfirmation',

    components: { LedgerConfirmationContent, TxConfirmation },

    props: {
        /** `accountInfo` object from `UnstakeFTM` component. */
        accountInfo: {
            type: Object,
            default() {
                return {};
            },
        },
        /***/
        stakerId: {
            type: String,
            default: '',
        },
    },

    data() {
        return {
            tx: {},
        };
    },

    computed: {
        ...mapGetters(['currentAccount']),
    },

    // activated() {
    mounted() {
        this.setTx();
    },

    methods: {
        async setTx() {
            this.tx = await this.$fWallet.getSFCTransactionToSign(
                sfcUtils.unstashRewardsTx(),
                this.currentAccount.address
            );
        },

        onSendTransactionSuccess(_data) {
            this.$emit('change-component', {
                to: 'transaction-success-message',
                from: 'unstash-confirmation',
                data: {
                    tx: _data.data.sendTransaction.hash,
                    successMessage: 'Unstashing Successful',
                    continueTo: 'staking-info',
                    continueToParams: {
                        stakerId: this.stakerId,
                    },
                },
            });
        },

        onBackBtnClick() {
            this.$emit('change-component', {
                to: 'staking-info',
                from: 'unstash-confirmation',
                data: {
                    stakerId: this.stakerId,
                },
                /*
                data: {
                    withdrawRequest: this.withdrawRequest,
                },
*/
            });
        },

        /**
         * Re-target `'change-component'` event.
         *
         * @param {object} _data
         */
        onChangeComponent(_data) {
            this.$emit('change-component', _data);
        },

        toFTM,
    },
};
</script>
