<template>
	<div>
		<el-row style="margin-bottom:10px;">
			<el-col :span="24">
				<div>
					<el-button type="primary" icon="el-icon-plus" @click="showAddDialog=true">添加轮播</el-button>
				</div>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData" style="width: 100%">
					<el-table-column prop="bid" label="轮播编号" width="180">
					</el-table-column>
					<el-table-column prop="imgpath" label="轮播图片" width="180">
						<!-- 图片的显示 -->
						<template slot-scope="scope">
							<img :src="CONSTANT.baseURL+scope.row.imgpath" min-width="70" height="70" />
						</template>
					</el-table-column>
					<el-table-column prop="url" label="跳转地址">
					</el-table-column>
					<el-table-column prop="status" label="状态">
						<template slot-scope="scope">
							<span v-if="scope.row.status==0">启用</span>
							<span v-if="scope.row.status==1">隐藏</span>
						</template>
					</el-table-column>
					<el-table-column fixed="right" label="操作" width="200">
						<template slot-scope="scope">
							<el-button size="mini" @click="selectById(scope.row.bid)">编辑</el-button>
							<el-button size="mini" type="danger" @click="deleteBanner(scope.row.bid)">删除</el-button>
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

		<el-dialog title="添加轮播" :visible.sync="showAddDialog">
			<el-form label-width="100px">
				<el-form-item label="轮播图片">
					<input type="file" class="form-control-file" @change="changeAddImage" />
				</el-form-item>
				<el-form-item label="跳转内容">
					<el-input v-model="add.url" placeholder="请输入跳转内容"></el-input>
				</el-form-item>
				<el-form-item label="轮播图状态">
					<el-select placeholder="请选择轮播图状态" v-model="add.status">
						<el-option label="显示" value="0"></el-option>
						<el-option label="隐藏" value="1"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showAddDialog = false">取 消</el-button>
				<el-button type="primary" @click="addBanner()">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="修改轮播" :visible.sync="showUpdateDialog">
			<el-form label-width="100px">
				<el-form-item label="轮播图片">
					<input type="file" class="form-control-file" @change="changeUpdateImage" />
				</el-form-item>
				<el-form-item label="跳转内容">
					<el-input v-model="update.url" placeholder="请输入跳转内容"></el-input>
				</el-form-item>
				<el-form-item label="轮播图状态">
					<el-select placeholder="请选择轮播图状态" v-model="update.status">
						<el-option label="显示" :value="0"></el-option>
						<el-option label="隐藏" :value="1"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showUpdateDialog = false">取 消</el-button>
				<el-button type="primary" @click="updateBanner()">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	export default {
		name: "bannermanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				showAddDialog: false,
				showUpdateDialog: false,
				option: '',
				pageinfo: {},
				add: {
					url: '',
					imgpath: '',
					status: "",
					imgFile: ''
				},
				update: {
					bid: 0,
					url: '',
					imgpath: '',
					status: "",
					imgFile: ''
				},
				dialogImageUrl: '',
				msg: ""
			}
		},
		mounted() {
			this.getBannerList();
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
			getBannerList() {
				this.axios.post("/admin/zbanner/getList", {
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
				this.axios.post("/admin/zbanner/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
			selectById(bid) {
				this.axios.post("/admin/zbanner/selectById?bid=" + bid, {
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
			 * 更改保存轮播图片框内容
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
			addBanner() {
				let data = new FormData();
				data.append("url", this.add.url);
				data.append("imgFile", this.add.imgFile);
				data.append("status", this.add.status);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zbanner/add", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.showAddDialog = false;
							this.msg = json.data.msg;
							this.open2();
							this.getBannerList();

						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {
								this.showAddDialog = false;
							})
						}
					})
			},
			/**
			 * 删除轮播
			 * @param bid
			 */
			deleteBanner(bid) {
				let flag = confirm("您确定要删除编号为[" + bid + "]的轮播信息吗?");
				if (flag) {
					this.axios.get("/admin/zbanner/delete?bid=" + bid)
						.then((json) => {
							console.log(json)
							if (json.data.code == "200") {
								this.msg = json.data.msg;
								//消息提示
								this.open2();
								//加载列表
								this.getBannerList();
							} else {}
						})
				}
			},
			/**
			 * 修改轮播
			 */
			updateBanner() {
				let data = new FormData();
				data.append("bid", this.update.bid);
				data.append("url", this.update.url);
				if (this.update.imgpath) {
					data.append("imgpath", this.update.imgpath);
				}
				data.append("imgFile", this.update.imgFile);
				data.append("status", this.update.status);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zbanner/update", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.update = {}
							this.msg = json.data.msg;
							this.showUpdateDialog = false;
							this.open2();
							this.getBannerList();
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
