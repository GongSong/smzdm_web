<template>
    <div class="slide" :style="bgColor">
        <div class="wrap">
            <div class="content" v-if="content!=''" v-html="content"></div>
            <div ref="chart" class="chart" :style="{height:chartHeight}"></div>
        </div>
        <arc-line v-if="arc" :fill="arc"></arc-line>
    </div>
</template>

<script>
import echarts from 'echarts';
import china from 'echarts/map/js/china';
import 'echarts-wordcloud';
// import 'echarts-gl';

import theme from '../lib/theme';
import defaultOption from '../lib/defaultOption';

export default {
    props: {
        option: {
            type: Object,
            default() {
                return defaultOption;
            }
        },
        fill: {
            default() {
                return false;
            }
        },
        arc: {
            default() {
                return false;
            }
        },
        content: {
            default() {
                return false;
            }
        }
    },
    mounted() {
        this.init();
    },
    computed: {
        chart() {
            return echarts.init(this.$refs.chart, theme);
        },
        chartHeight() {
            return window.innerHeight * 0.9 + 'px';
        },
        bgColor() {
            switch (this.fill) {
                case true:
                case 'true':
                    return {
                        backgroundColor: '#f5f5f5'
                    };
                case false:
                case 'false':
                    return {};
                default:
                    return {
                        backgroundColor: this.fill
                    };
            }
        }
    },
    methods: {
        resizeChart() {
            this.chart.resize();
        },
        initEvent() {
            window.onresize = () => {
                this.resizeChart();
            }
        },
        init() {
            this.initChart();
            this.initEvent();
        },
        initChart() {
            // 文字云导出时，自定义函数无法输出
            if (typeof this.option.series == 'object' && this.option.series.type == 'wordCloud') {
                this.option.series.textStyle = {
                    normal: {
                        color: param => theme.color[param.dataIndex % theme.color.length]
                    }
                };
            }

            if (this.fill) {
                this.option.backgroundColor = this.bgColor;
            }

            this.option.toolbox = {
                show: false
            };

            if (Reflect.has(this.option, 'tooltip')) {
                if (Reflect.has(this.option.tooltip, 'axisPointer')) {
                    if (this.option.tooltip.axisPointer.type == 'line') {
                        this.option.tooltip.axisPointer = {
                            type: 'cross',
                            lineStyle: {
                                type: 'dashed'
                            }
                        };
                    }
                }
            }
            if (typeof this.option.title == 'object') {
                this.option.title.top = 0;
            } else {
                this.option.title[0].top = 0;
            }
            if (Reflect.has(this.option, 'xAxis')) {
                this.option.xAxis[0].boundaryGap = true;
            }
            this.chart.setOption(this.option);
        }
    }
};
</script>
<style scoped>
.chart {
    width: 90%;
    height: 90%;
}

.content {
    width: 100%;
}
</style>