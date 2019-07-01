<template>
	<div>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData" style="width: 100%">
					<el-table-column prop="eid" label="评价编号" width="180">
					</el-table-column>
					<el-table-column prop="imgpath" label="评价图片" width="180">
						<!-- 图片的显示 -->
						<template slot-scope="scope">
							<img :src="CONSTANT.baseURL+scope.row.imgpath" min-width="70" height="70" />
						</template>
					</el-table-column>
					<el-table-column prop="content" label="评价内容">
					</el-table-column>
					<el-table-column prop="zgoods.gname" label="商品名称">
					</el-table-column>
					<el-table-column prop="zuser.username" label="用户名">
					</el-table-column>
					<el-table-column prop="star" label="评分">
					</el-table-column>
					
				</el-table>
			</el-col>
		</el-row>

		<el-row style="margin-top:10px;">
			<el-button type="primary" @click="getPage(1)">首页</el-button>
			<el-button type="success" @click="getPage(pageinfo.prevpage)">上一页</el-button>
			<el-button type="success" @click="getPage(pageinfo.nextpage)">下一页</el-button>
			<el-button type="primary" @click="getPage(pageinfo.pagecount)">尾页</el-button>
		</el-row>


	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	export default {
		name: "evaluationmanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				pageinfo: {},
			}
		},
		mounted() {
			this.getEvaluationList();
		},
		methods: {
			/**
			 * 加载列表
			 */
			getEvaluationList() {
				this.axios.post("/admin/zEvaluation/getList", {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data
						this.pageinfo = json.data.pageBean;
					})
			},
			/**
			 * @param {Object} pageIndex获取分页数据
			 */
			getPage(pageIndex) {
				this.axios.post("/admin/zEvaluation/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
		}
	}
</script>

<style scoped>

</style>
