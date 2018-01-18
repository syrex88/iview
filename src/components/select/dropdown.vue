<template>
    <div class="ivu-select-dropdown" :class="className" :style="styles">
        <slot></slot>
    </div>
</template>
<script>
    import Vue from 'vue';

    const isServer = Vue.prototype.$isServer;
    import {getStyle} from '../../utils/assist';

    const Popper = isServer ? function () {
    } : require('popper.js');  // eslint-disable-line

    export default {
        name: 'Drop',
        props: {
            placement: {
                type: String,
                default: 'bottom-start'
            },
            dropDownLeftOffset: { // todo не работает передача свойства
                type: String,
                default: '0'
            },
            className: {
                type: String
            }
        },
        data() {
            return {
                popper: null,
                width: ''
            };
        },
        computed: {
            styles() {
                let style = {};
                if (this.width) style.minWidth = `${this.width + 10}px`; // todo хардкод, сделать автовычисление с поправкой на padding
                return style;
            }
        },
        methods: {
            update() {
                if (isServer) return;
                if (this.popper) {
                    this.$nextTick(() => {
                        this.popper.update();
                        this.popper._popper.style.left = this.dropDownLeftOffset + 'px'; // todo проверить при ресайзе окна браузера
                    });
                } else {
                    this.$nextTick(() => {
                        this.popper = new Popper(this.$parent.$refs.reference, this.$el, {
                            gpuAcceleration: false,
                            placement: this.placement,
                            boundariesPadding: 0,
                            forceAbsolute: true,
                            boundariesElement: 'body'
                        });
                        this.popper.onCreate(popper => {
                            this.resetTransformOrigin(popper);
                        });
                    });
                }
                // set a height for parent is Modal and Select's width is 100%
                if (this.$parent.$options.name === 'iSelect') {
                    this.width = parseInt(getStyle(this.$parent.$el, 'width'));
                }
            },
            destroy() {
                if (this.popper) {
                    this.resetTransformOrigin(this.popper);
                    setTimeout(() => {
                        this.popper.destroy();
                        this.popper = null;
                    }, 300);
                }
            },
            resetTransformOrigin(popper) {
                let placementMap = {top: 'bottom', bottom: 'top'};
                let placement = popper._popper.getAttribute('x-placement').split('-')[0];
                let origin = placementMap[placement];
                popper._popper.style.transformOrigin = `center ${ origin }`;
                popper._popper.style.left = this.dropDownLeftOffset + 'px';
            }
        },
        created() {
            this.$on('on-update-popper', this.update);
            this.$on('on-destroy-popper', this.destroy);
        },
        beforeDestroy() {
            if (this.popper) {
                this.popper.destroy();
            }
        }
    };
</script>
