<template>
    <div v-if="localShow"
         :class="classObject"
         role="alert"
         aria-live="polite"
         aria-atomic="true"
    >
        <button type="button"
                class="close"
                data-dismiss="alert"
                :aria-label="dismissLabel"
                v-if="localDismissible"
                @click.stop.prevent="dismiss"
        >
            <span aria-hidden="true">&times;</span>
        </button>
        <slot></slot>
    </div>
</template>

<script>
    import {warn} from '../utils';

    export default {
        data() {
            return {
                countDownTimerId: null,
                dismissed: false,
                localDismissible: this.dismissible
            };
        },
        created() {
            if (this.state) {
                warn('<b-alert> "state" property is deprecated, please use "variant" property instead.');
            }
        },
        computed: {
            classObject() {
                return ['alert', this.alertVariant, this.localDismissible ? 'alert-dismissible' : ''];
            },
            alertVariant() {
                const variant = this.state || this.variant || 'info';
                return `alert-${variant}`;
            },
            localShow() {
                return !this.dismissed && (this.countDownTimerId || this.show);
            }
        },
        props: {
            variant: {
                type: String,
                default: 'info'
            },
            state: {
                type: String,
                default: null
            },
            dismissible: {
                type: Boolean,
                default: false
            },
            dismissLabel: {
                type: String,
                default: 'Close'
            },
            show: {
                type: [Boolean, Number],
                default: false
            }
        },
        watch: {
            show() {
                this.showChanged();
            }
        },
        mounted() {
            this.showChanged();
        },
        methods: {
            dismiss() {
                this.dismissed = true;
                this.$emit('dismissed');
                this.clearCounter();
            },
            clearCounter() {
                if (this.countDownTimerId) {
                    clearInterval(this.countDownTimerId);
                }
            },
            showChanged() {
                // Reset dismiss status
                this.dismissed = false;

                // No timer for boolean values
                if (this.show === true || this.show === false || this.show === null || this.show === 0) {
                    this.localDismissible = this.dismissible;
                    return;
                }

                // Hide dismiss button for auto-dismissing
                this.localDismissible = false;

                let dismissCountDown = this.show;
                this.$emit('dismiss-count-down', dismissCountDown);

                // Start counter
                this.clearCounter();
                this.countDownTimerId = setInterval(() => {
                    if (dismissCountDown < 2) {
                        return this.dismiss();
                    }
                    dismissCountDown--;
                    this.$emit('dismiss-count-down', dismissCountDown);
                }, 1000);
            }
        }
    };
</script>
