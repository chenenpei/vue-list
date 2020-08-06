<template>
    <div class="app-my-modal" v-if="show">
        <div class="mask" :class="{ visible: showEffect }" @click="onMaskClick"></div>

        <div class="content-wrapper" :class="{ visible: showEffect }">
            <header v-if="showTitle">
                <slot name="title">
                    <p>这是个简简单单的标题</p>
                </slot>
            </header>
            
            <slot>something</slot>
            
            <footer v-if="showActions" class="actions">
                <slot name="actions">
                    <button style="margin-right: 8px" @click="onClose">{{cancelText}}</button>
                    <button @click="onOk">{{confirmText}}</button>
                </slot>
            </footer>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        visible: {
            type: Boolean,
            default: false,
        },
        confirmText: {
            type: String,
            default: "确认",
        },
        cancelText: {
            type: String,
            default: "取消",
        },
        onClose: {
            type: Function,
            default: () => {},
        },
        onOk: {
            type: Function,
            default: () => {},
        },
        // 是否展示标题
        showTitle: {
            type: Boolean,
            default: true,
        },
        // 是否显示弹窗底部按钮
        showActions: {
            type: Boolean,
            default: true
        },
    },
    data() {
        return {
            show: false,
            showEffect: false,
        };
    },
    watch: {
        visible(val) {
            if (val) {
                this.show = val;

                // 下一次渲染再附上动效
                setTimeout(() => {
                    this.showEffect = val;
                }, 0);
            } else {
                this.showEffect = val;

                // 动效完成后再隐藏弹窗（动效暂定 500ms）
                setTimeout(() => {
                    this.show = false;
                }, 500);
            }
        },
    },
    methods: {
        /**
         * 点击蒙层触发的回调
         */
        onMaskClick() {
            this.$emit("mask-click");
        },
    },
};
</script>

<style lang="scss">
.app-my-modal {
    .mask {
        position: fixed;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        background: rgba(0, 0, 0, 0.6);
        z-index: 200;
        transition: opacity ease 0.5s;
        opacity: 0;
        &.visible {
            opacity: 1;
        }
    }
    .content-wrapper {
        padding: 16px;
        position: fixed;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        background: #fff;
        border-radius: 4px;
        z-index: 201;
        transform: scale(0) translate(-50%, -50%);
        transform-origin: top left;
        transition: transform ease 0.5s;
        &.visible {
            transform: scale(1) translate(-50%, -50%);
        }
        .actions {
            margin-top: 16px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
    }
}
</style>
