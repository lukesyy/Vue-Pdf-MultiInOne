<!--
 * @Date: 2021-11-09 10:10:51
 * @LastEditors: LiJinGang
 * @LastEditTime: 2021-11-09 12:10:51
-->
<template>
	<div id="app">
		<div class="box">
			<div class="title">弹窗预览并下载</div>
			<PDF />
		</div>
		<div class="box">
			<div class="title">无预览 静默下载</div>
			<PDFF />
		</div>
	</div>
</template>

<script>
import PDF from './components/PdfMultiInOne.vue'
import PDFF from './components/noDialog.vue'

export default {
	components: {
		PDF,
		PDFF
	}
}
</script>
<style scope>
#app {
	text-align: center;
	margin: 0 auto;
}
.box {
	padding: 50px;
}
.title {
	font-size: 24px;
	color: coral;
  padding-bottom: 10px;
}
</style>
