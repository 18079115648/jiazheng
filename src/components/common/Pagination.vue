<template>
    <mt-loadmore :bottom-method="loadBottom" :top-method="loadTop" :auto-fill="false" :bottom-all-loaded="allLoaded" :topDistance="40" :bottomDistance="40" ref="loadmore">
        <slot></slot>
    </mt-loadmore>
</template>

<script>
import {Loadmore, Indicator} from 'mint-ui'
export default {
    props: {
        render: {
            type: Function,
            required: true
        },
        param: {
            type: Object,
            required: true
        },
        needToken: {
            type: Boolean,
            default: true
        },
        uri: {
            type: String,
            required: true
        },
        autoload: {
            type: Boolean,
            default: true,
            required: false
        }
    },
    data() {
        return {
            allLoaded: false
        }
    },
    mounted() {
        if (this.autoload) {
            this.loadPage()
        }
    },
    methods: {
        // 下拉分页加载
        loadBottom() {
            this.loadPage(false)
        },
        // 上拉刷新加载
        loadTop() {
            this.param.page = 1
//          this.param.content = []
            this.loadPage(true)
        },
        // 刷新
        refresh() {
            this.loadPage(true)
        },
        // 加载分页
        loadPage(isRefresh) {
            let self = this
            self.param.loadEnd = false
            Indicator.open('加载中...')
            setTimeout(function() {
                self.$http.page(self.needToken, self.uri, self.param, function(response) {
                	Indicator.close()
                    if (isRefresh) {
                        self.param.page = 1
                        self.param.content = []
                        self.allLoaded = false
                    }

                    self.render(response)
					self.param.loadEnd = true
                    if (response.result.page >= response.result.total_pages || response.result.total_pages === 0 ) {
                        // 设置加1
                        self.allLoaded = true
                    } else {
                        self.param.page++
                    }

                    if (isRefresh) {
                        self.$refs.loadmore.onTopLoaded()
                    } else {
                        self.$refs.loadmore.onBottomLoaded()
                    }
                }, self.param.param)
            }, 200)
        }
    }
}
</script>

<style scoped>

</style>


