<template>
    <div class="app-vue-list" :style="{ height: number * height + 'px' }" @scroll="handleScroll">
        <ul :style="{ paddingTop: lineTopHeight +'px', paddingBottom: lineBottomHeight +'px' }">
            <li
                v-for="(item, index) in previewList"
                :key="index"
                :style="{ height: height + 'px' }"
            >
                <slot name="listItem" :data="{ index, item }">{{ item }}</slot>
            </li>
            <li>Loading...</li>
        </ul>
    </div>
</template>

<script>
export default {
    name: "Vuelist",
    props: {
        // 项目列表
        list: {
            type: Array,
            required: true,
            default() {
                return [];
            }
        },
        // 默认当个项目的高度
        height: {
            type: Number,
            default: 44
        },
        // 默认单屏项目的个数
        number: {
            type: Number,
            default: 10
        },
        canScroll: {
            type: Boolean,
            default: true
        }
    },
    data() {
        return {
            previewList: [],
            lastScrollTop: null,
            canLoadMore: true,
            lineTopHeight: 0,
            lineBottomHeight: 0
        };
    },
    watch: {
        list(val) {
            // 如果传入的列表发生改变，需要重新组织 previewList
            // 常见于搜索列表的场景
            this.previewList = val.slice(0, this.number);
        }
    },
    mounted() {
        this.initData();
        this.handleScroll();
    },
    methods: {
        initData() {
            this._rowsInWindow = Math.floor(
                this.$el.offsetHeight / this.height
            );
            this._above = this._rowsInWindow * 2;
            this._below = this._rowsInWindow;
            this._max =
                Math.ceil(this.$el.offsetHeight / this.height) * this.height;
        },
        handleScroll() {
            let totalHeight = this.$el.querySelector("ul").offsetHeight;
            let contentHeight = this.$el.offsetHeight;
            let scrollTop = this.$el.scrollTop;

            // 当屏幕的 scrollTop 减去上一次的 scrollTop 会大于可见项的总高度的时候，重组列表数据
            if (
                this.lastScrollTop === null ||
                Math.abs(scrollTop - this.lastScrollTop) > this._max
            ) {
                this.lastScrollTop = scrollTop;

                // 计算截取的开始项和结束项
                // 算出应该从列表数据第几个开始截取（也就是超出了 above 高度的项的个数）
                let from = parseInt(scrollTop / this.height) - this._above;
                if (from < 0) from = 0;
                let to = from + this._above + this._rowsInWindow + this._below;
                if (to > this.list.length) to = this.list.length;
                this.from = from;
                this.to = to;

                this.lineTopHeight = from * this.height;
                this.lineBottomHeight = (this.list.length - to) * this.height;

                this.resetPreviewList(from, to);

                this.$nextTick(() => {
                    let totalHeight = this.$el.querySelector("ul").offsetHeight;
                    let contentHeight = this.$el.offsetHeight;
                    let scrollTop = this.$el.scrollTop;

                    if (
                        to === this.list.length &&
                        totalHeight - scrollTop - contentHeight < 0
                    ) {
                        this.canScroll && this.loadMore(this.from, this.to);
                    }
                });

                return;
            }

            // 加载数据
            if (
                this.to === this.list.length &&
                totalHeight - scrollTop - contentHeight < this.height
            ) {
                this.canScroll && this.loadMore(this.from, this.to);
            }
        },
        /**
         * 加载数据
         * @param from
         * @param to
         */
        loadMore(from, to) {
            if (!this.canLoadMore) return;
            this.canLoadMore = false;

            //  模拟获取数据
            setTimeout(() => {
                this.$emit("load");
                this.$nextTick(() => {
                    this.resetPreviewList(from, to + this._below);
                });
                this.lineBottomHeight = (this.list.length - to) * this.height;
                this.handleScroll();
                this.canLoadMore = true;
            }, 1000);
        },
        /**
         * 重组展示列表
         * @param from
         * @param to
         */
        resetPreviewList(from, to) {
            this.previewList = [];
            while (from < to) {
                this.previewList.push(this.list[from]);
                from++;
            }
        }
    }
};
</script>

<style>
.app-vue-list {
    width: 100%;
    overflow-y: auto;
}
.app-vue-list > ul {
    margin: 0;
    padding: 0;
}
.app-vue-list > ul > li {
    box-sizing: border-box;
    padding-left: 15px;
    font-size: 14px;
    line-height: 3;
    text-align: left;
    text-decoration: none;
    border-bottom: 1px solid #ddd;
    background: #fff;
}
</style>
