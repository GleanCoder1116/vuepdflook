<template>
	<div v-show="isready" class="cpdf">
		<div class="center">
			<div class="contor">
				<button class="btn" @click="prev">
					<span>上一页</span>
				</button>
				<button class="btn" @click="next">下一页</button> &nbsp; &nbsp;
				<span>Page: <span v-text="page_num"></span> / <span v-text="page_count"></span></span>
				&nbsp; &nbsp;
				<button class="btn" @click="addscale" >+</button>
				<button class="btn" @click="minus" >-</button>
				<button class="btn" @click="closepdf">关闭</button>
				
			</div>
			<canvas class="canvasstyle" id="the-canvas"></canvas>
		</div>
	</div>
</template>
<script>
    import PDFJS from './pdf/pdf.js';

    export default {
        name: 'c-pdf',
        props: {
            pdfurl: {
                default: '',
            },
            value: {

            },
            maxscale: {
                default: 2,
                type: Number
            },
            minscale: {
                default: 0.8,
                type: Number
            },
            scale: {
                default: 1.2,
                type: Number
            }
        },
        //['pdfurl', 'value'],
        data() {

            return {
                pdfDoc: null, //pdfjs 生成的对象
                pageNum: 1, //
                pageRendering: false,
                pageNumPending: null,
                compuscale: 0, //放大倍数
                page_num: 0, //当前页数
                page_count: 0, //总页数
                // maxscale: 2, //最大放大倍数
                // minscale: 0.8, //最小放大倍数
                isready: false
            }
        },
        methods: {
            renderPage(num) { //渲染pdf
                let vm = this
                this.pageRendering = true;
                let canvas = document.getElementById('the-canvas')
                    // Using promise to fetch the page
                this.pdfDoc.getPage(num).then(function(page) {
                    var viewport = page.getViewport(vm.compuscale);
                    //alert(vm.canvas.height)
                    canvas.height = viewport.height;
                    canvas.width = viewport.width;

                    // Render PDF page into canvas context
                    vm.isready = true
                    var renderContext = {
                        canvasContext: vm.ctx,
                        viewport: viewport
                    };
                    var renderTask = page.render(renderContext);

                    // Wait for rendering to finish
                    renderTask.promise.then(function() {
                        vm.pageRendering = false;
                        if (vm.pageNumPending !== null) {
                            // New page rendering is pending
                            vm.renderPage(vm.pageNumPending);
                            vm.pageNumPending = null;
                        }
                    });
                });
                vm.page_num = vm.pageNum;

            },
            addscale() { //放大
                if (this.compuscale >= this.maxscale) {
                    return
                }
                this.compuscale += 0.1;
                this.queueRenderPage(this.pageNum)
            },
            minus() { //缩小
                if (this.compuscale <= this.minscale) {
                    return
                }
                this.compuscale -= 0.1;
                this.queueRenderPage(this.pageNum)
            },
            prev() { //上一页
                let vm = this
                if (vm.pageNum <= 1) {
                    return;
                }
                vm.pageNum--;
                vm.queueRenderPage(vm.pageNum);
            },
            next() { //下一页
                let vm = this
                if (vm.pageNum >= vm.page_count) {
                    return;
                }
                vm.pageNum++;
                vm.queueRenderPage(vm.pageNum);
            },
            closepdf() { //关闭PDF
                this.pdfDoc = null;
                this.isready = false
                this.compuscale = this.scale
                this.$emit('closepdf')
            },
            throwerr(err) {
                this.$emit('pdferr', err);
            },
            queueRenderPage(num) {
                if (this.pageRendering) {
                    this.pageNumPending = num;
                } else {
                    this.renderPage(num);
                }
            }
        },
        computed: {
            ctx() {
                let id = document.getElementById('the-canvas')
                return id.getContext('2d');
            },
            isshowpdf() {
                if (this.value == undefined) return false
                return this.value;
            }
        },
        mounted() {
            this.compuscale = this.scale
        },
        watch: {
            isshowpdf(b) {
                let vm = this
                var val = vm.pdfurl
                    //console.log(b)

                if (b == false || val == '' || val == undefined) return
                PDFJS.getDocument(val).then(function(pdfDoc_) { //初始化pdf
                    vm.pdfDoc = pdfDoc_;
                    vm.page_count = vm.pdfDoc.numPages
                    vm.renderPage(vm.pageNum);
                }).catch(function(err) {
                    if (err) {
                        console.log(err)
                        vm.throwerr(vm.pdfurl)
                    }
                })
            }


        }
    }
</script>

<style>
    .cpdf {
        position: fixed;
        top: 0;
        left: 0;
        background-color: rgba(0, 0, 0, .5);
        width: 100%;
        height: 100%;
        z-index: 99999;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    
    .cpdf .btn {
        font-size: 12px;
        display: inline-block;
        line-height: 1;
        white-space: nowrap;
        cursor: pointer;
        background: #fff;
        border: 1px solid #bfcbd9;
        color: #1f2d3d;
        -webkit-appearance: none;
        text-align: center;
        box-sizing: border-box;
        outline: 0;
        margin: 0;
        padding: 8px 10px;
        border-radius: 4px;
    }
    
    .cpdf .center {
        text-align: center;
        height: 100%;
        overflow: auto;
        padding-top: 20px;
    }
    
    .cpdf .center .contor {
        margin-bottom: 10px;
    }
</style>