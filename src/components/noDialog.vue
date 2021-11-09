<template>
	<div>
		<el-button type="primary" @click="openPdf('a4')">点击下载pdf多合一(A4)</el-button>
		<el-button type="primary" @click="openPdf('a5')">点击下载pdf多合一(A5)</el-button>
		<div id="pop" v-if="show">
			<template v-for="item in pdfUrlArr">
				<pdf :src="item.pdfUrl" v-for="i in item.page" :key="i + numPagesAll - 1" :page="i"></pdf>
			</template>
		</div>
	</div>
</template>
<script src="./js/jsPdf.debug.js"></script>
<script>
import pdf from 'vue-pdf'

export default {
	components: {
		pdf
	},
	data() {
		return {
			lodDisabled: true,
			pdfUrlArr: [],
			numPages: null,
			pdfUrl: '',
			numPagesAll: 0,
			timer: null,
			show: false,
			tag: 'a4'
		}
	},
	methods: {
		initData() {
			this.numPages = null
			this.pdfUrl = ''
			this.numPagesAll = 0
		},
		openPdf(tag) {
			this.initData()
			this.tag = tag
			this.lodDisabled = true
			// this.pdfUrlArr = 你需要合成的pdf链接就可以了 但是 结构必须和我示例的相同  无视跨域
			this.pdfUrlArr = [
				{ pdfUrl: 'https://lawkehyt.oss-cn-beijing.aliyuncs.com/%E5%BC%A0%E5%96%9C%E4%B9%90_eb5e6.pdf', page: 0 },
				{ pdfUrl: 'https://lawkehyt.oss-cn-beijing.aliyuncs.com/%E6%9C%B1%E4%BA%91%E4%B8%AD_bbefa.pdf', page: 0 },
				{ pdfUrl: 'https://lawkehyt.oss-cn-beijing.aliyuncs.com/%E9%AB%98%E9%B9%8F%E9%A3%9E_b4bce.pdf', page: 0 }
			]
			this.show = true
			this.previewPDF()
		},
		//预览合并pdf
		async previewPDF() {
			let PromiseArr = []
			this.pdfUrlArr.forEach(item => {
				PromiseArr.push(
					pdf.createLoadingTask(item.pdfUrl, { cMapUrl: 'https://cdn.jsdelivr.net/npm/pdfjs-dist@2.9.359/cmaps/', cMapPacked: true }).promise
				)
			})
			let res = await Promise.all(PromiseArr)

			res.forEach((item, index) => {
				this.numPagesAll += item.numPages
				this.pdfUrlArr[index].page = item.numPages
			})
			this.timer = setInterval(() => {
				let canvasList = document.querySelectorAll('canvas')
				if (this.numPagesAll <= canvasList.length) {
					if (
						// 此处判断canvas是否已经全部绘制完成
						canvasList[canvasList.length - 1].toDataURL('image/JPEG') !=
						'data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAMCAgICAgMCAgIDAwMDBAYEBAQEBAgGBgUGCQgKCgkICQkKDA8MCgsOCwkJDRENDg8QEBEQCgwSExIQEw8QEBD/2wBDAQMDAwQDBAgEBAgQCwkLEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBD/wAARCACWASwDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAn/xAAUEAEAAAAAAAAAAAAAAAAAAAAA/8QAFAEBAAAAAAAAAAAAAAAAAAAAAP/EABQRAQAAAAAAAAAAAAAAAAAAAAD/2gAMAwEAAhEDEQA/AJVAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//9k='
					) {
						clearInterval(this.timer)
						this.lodDisabled = false
						this.createPDF()
					}
				}
			}, 500)
		},
		createPDF() {
			let RL = this.tag == 'a4' ? '' : 'l'
			let canvasList = document.querySelectorAll('canvas')
			var doc = new jsPDF(RL, 'pt', this.tag)
			canvasList.forEach((canvas, index) => {
				if (index > 0) doc.addPage() //添加页
				//a4纸的尺寸[595.28,841.89]，html页面生成的canvas在pdf中图片的宽高
				var contentWidth = canvas.width
				var contentHeight = canvas.height
				var imgWidth = 595.28
				var imgHeight = (592.28 / contentWidth) * contentHeight
				var canvas_1Url = canvas.toDataURL('image/JPEG')
				doc.addImage(canvas_1Url, 'JPEG', 0, 0, imgWidth, imgHeight)
			})
			// 自行更改要保存的pdf名字
			doc.save('合并pdf.pdf')
      this.show = false
		}
	},
	mounted() {}
}
</script>

<style scope>
body{
  overflow: hidden;
}
#pop {
	position: absolute;
	left: 100%;
	width: 1920px;
  overflow: visible;
}
.el-dialog__wrapper .el-dialog {
	margin-top: 2vh !important;
}
</style>
