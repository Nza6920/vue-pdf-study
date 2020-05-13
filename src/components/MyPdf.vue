<template>
    <div class="canvas-container">
        <canvas ref="myCanvas" class="pdf-container">
        </canvas>
    </div>
</template>
<script>
    import pdfJS from 'pdfjs-dist'

    // 引入pdf.js的字体
    const CMAP_URL = '../assets/cmaps';
    export default {
        name: "MyPdf",
        props: {
            pageNumber: {
                type: Number, // 页数
                default: function () {
                    return 1
                }
            },
            scale: { // 缩放值
                type: Number,
                default: function () {
                    return 1
                }
            },
            data: { // PDF base64编码
                type: String,
                default: function () {
                    return ''
                }
            }
        },
        data() {
            return {
                loadFinished: false
            }
        },
        watch: {
            pageNumber() { // 监听页数改变
                this.previewPDF()
            },
            data(newVal) {
                if (newVal) {
                    this.previewPDF()
                }
            }
        },
        mounted() {
            this.previewPDF()
        },
        methods: {
            previewPDF() {
                const _this = this
                if (!_this.data) {
                    return;
                }
                _this.loadFinished = false
                let loadingTask = pdfJS.getDocument({
                    data: _this.data, // PDF base64编码
                    cMapUrl: CMAP_URL,
                    cMapPacked: true
                })
                loadingTask.promise.then((pdf) => {
                    _this.getPage(pdf, _this.pageNumber)
                })
            },
            getPage(pdf, pageNumber) {  // 获取pdf指定页
                let _this = this
                if (pageNumber <= 0 || pageNumber > pdf.numPages) { // 判断页数是否超过
                    _this.$emit('overflow', {max: pdf.numPages})
                } else {
                    pdf.getPage(pageNumber)
                        .then((page) => {
                            // 获取canvasDOM对象
                            let canvas = _this.$refs.myCanvas
                            let viewport = page.getViewport(_this.scale)
                            canvas.height = viewport.height
                            canvas.width = viewport.width

                            let ctx = canvas.getContext('2d')
                            let renderContext = {
                                canvasContext: ctx,
                                viewport: viewport
                            }
                            // 显示pdf
                            page.render(renderContext).then(() => {
                                _this.$emit('loadFinished')
                                _this.loadFinished = true
                            })
                        })
                }
            }
        }
    }
</script>

<style scoped>
    .pdf-container {
        width: 800px;
    }

    .canvas-container {
        width: 900px;
        margin: 0 auto;
        text-align: center;
        border: 1px dashed black;
        position: relative;
    }
</style>
