<template>
    <div id="app">
        <input type="file" ref="fielinput" @change="uploadFile"/>
        <button v-if="showPdf" @click="prePage">上一页</button>
        <button v-if="showPdf" @click="nextPage">下一页</button>
        <my-pdf v-if="showPdf" :page-number="curPage" :scale="2" :data="pdfData" @overflow="overflow" @loadFinished="loadFinished"/>
    </div>
</template>

<script>
    import MyPdf from './components/MyPdf'

    export default {
        name: 'App',
        components: {MyPdf},
        data() {
            return {
                pdfData: '', // base64
                showPdf: false,
                curPage: 1
            }
        },
        methods: {
            uploadFile() {
                this.showPdf = false
                let inputDom = this.$refs.fielinput
                let file = inputDom.files[0]
                let reader = new FileReader()
                reader.readAsDataURL(file)
                reader.onload = () => {
                    this.pdfData = atob(reader.result.substring(reader.result.indexOf(',') + 1))
                    this.showPdf = true
                }
            },
            overflow() {
                this.curPage = 1
            },
            loadFinished() {
                console.log('finished')
            },
            nextPage() {
                this.curPage ++
            },
            prePage() {
                this.curPage --
            }
        }
    }
</script>

<style>

</style>
