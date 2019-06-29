<template>
	<div>
		<el-row style="margin-bottom:10px;">
			<el-col :span="24">
				<div>
					<el-button type="primary" icon="el-icon-plus" @click="showAddDialog=true">添加商品类型</el-button>
				</div>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData" style="width: 100%">
					<el-table-column prop="gtypeid" label="商品类型编号"></el-table-column>
					<el-table-column prop="gtypename" label="商品类型名称" width="180">
					</el-table-column>
					<el-table-column prop="iconimgpath" label="商品类型图片" width="180">
						<!-- 图片的显示 -->
						<template slot-scope="scope">
							<img :src="CONSTANT.baseURL+scope.row.iconimgpath" min-width="70" height="70" />
						</template>
					</el-table-column>
					<el-table-column prop="description" label="商品类型简介">
					</el-table-column>
					<el-table-column fixed="right" label="操作" width="200">
						<template slot-scope="scope">
							<el-button size="mini" @click="selectById(scope.row.gtypeid)">编辑</el-button>
							<el-button size="mini" type="danger" @click="deleteGoodType(scope.row.gtypeid)">删除</el-button>
						</template>
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

		<el-dialog title="添加商品类型" :visible.sync="showAddDialog">
			<el-form label-width="100px">
				<el-form-item label="商品类型图片">
					<input type="file" class="form-control-file" @change="changeAddImage" />
				</el-form-item>
				<el-form-item label="商品类型名称">
					<el-input v-model="add.gtypename" placeholder="请输入商品类型名称"></el-input>
				</el-form-item>
				<el-form-item label="商品类型简介">
					<el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="商品类型简介" v-model="add.description">
					</el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showAddDialog = false">取 消</el-button>
				<el-button type="primary" @click="addGoodType()">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="修改商品类型" :visible.sync="showUpdateDialog">
			<el-form label-width="100px">
				<el-form-item label="商品类型图片">
					<input type="file" class="form-control-file" @change="changeUpdateImage" />
				</el-form-item>
				<el-form-item label="商品类型名称">
					<el-input v-model="update.gtypename" placeholder="请输入商品类型名称"></el-input>
				</el-form-item>
				<el-form-item label="商品类型简介">
					<el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="商品类型简介" v-model="update.description">
					</el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showUpdateDialog = false">取 消</el-button>
				<el-button type="primary" @click="updateGoodType()">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	export default {
		name: "goodtypemanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				showAddDialog: false,
				showUpdateDialog: false,
				option: '',
				pageinfo: {},
				add: {
					gtypename: '',
					iconimgpath: '',
					description: "",
					imgFile: ''
				},
				update: {
					gtypeid: 0,
					gtypename: '',
					iconimgpath: '',
					description: "",
					imgFile: ''
				},
				dialogImageUrl: '',
				msg: ""
			}
		},
		mounted() {
			this.getGoodTypeList();
		},
		methods: {
			/**
			 * 表格编辑按钮
			 * @param index
			 * @param row
			 */
			handleEdit(index, row) {
				this.showUpdateDialog = true;
			},
			/**
			 * 表格删除按钮
			 * @param index
			 * @param row
			 */
			handleDelete(index, row) {
				console.info(index);
				console.info(row);
			},
			/**
			 * 加载列表
			 */
			getGoodTypeList() {
				this.axios.post("/admin/goodsType/getList", {
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
				this.axios.post("/admin/goodsType/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
			selectById(gtypeid) {
				this.axios.post("/admin/goodsType/selectById?gtypeid=" + gtypeid, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.update = json.data.data;
						this.showUpdateDialog = true;
					})
			},
			/**
			 * 更改保存商品类型图片框内容
			 * @param e
			 */
			changeAddImage(e) {
				this.add.imgFile = e.target.files[0]
			},
			/**
			 * 修改模态框更换图片
			 */
			changeUpdateImage(e) {
				this.update.imgFile = e.target.files[0]
			},
			/**
			 * 新增
			 */
			addGoodType() {
				let data = new FormData();
				data.append("imgFile", this.add.imgFile);
				data.append("gtypename", this.add.gtypename);
				data.append("description", this.add.description);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/goodsType/add", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.showAddDialog = false;
							this.msg = json.data.msg;
							this.open2();
							this.getGoodTypeList();

						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {
								this.showAddDialog = false;
							})
						}
					})
			},
			/**
			 * 删除商品类型
			 * @param gtypeid
			 */
			deleteGoodType(gtypeid) {
				let flag = confirm("您确定要删除编号为[" + gtypeid + "]的商品类型信息吗?");
				if (flag) {
					this.axios.get("/admin/goodsType/delete?gtypeid=" + gtypeid)
						.then((json) => {
							console.log(json)
							if (json.data.code == "200") {
								this.msg = json.data.msg;
								//消息提示
								this.open2();
								//加载列表
								this.getGoodTypeList();
							} else {}
						})
				}
			},
			/**
			 * 修改商品类型
			 */
			updateGoodType() {
				let data = new FormData();
				data.append("gtypeid", this.update.gtypeid);
				if (this.update.iconimgpath) {
					data.append("iconimgpath", this.update.iconimgpath);
				}
				data.append("imgFile", this.update.imgFile);
				data.append("gtypename", this.update.gtypename);
				data.append("description", this.update.description);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/goodsType/update", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.update = {}
							this.msg = json.data.msg;
							this.showUpdateDialog = false;
							this.open2();
							this.getGoodTypeList();
						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {
								jQuery('#updateModal').modal('hide');
							})
						}
					})
			},
			/**
			 * 消息提示
			 */
			open2() {
				this.$message({
					message: this.msg,
					type: 'success'
				});
			},
		}
	}
</script>

<style scoped>

</style>
