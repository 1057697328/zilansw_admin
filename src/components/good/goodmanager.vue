<template>
	<div>
		<el-row style="margin-bottom:10px;">
			<el-col :span="24">
				<div>
					<el-button type="primary" icon="el-icon-plus" @click="OpenAddModel()">添加商品</el-button>
				</div>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData.filter(data => !search || data.gname.toLowerCase().includes(search.toLowerCase()))"
				 style="width: 100%">
					<el-table-column prop="gid" label="商品编号"></el-table-column>
					<el-table-column prop="goodstype.gtypename" label="商品类型名称">
					</el-table-column>
					<el-table-column prop="gname" label="商品名称">
					</el-table-column>
					<!-- <el-table-column prop="gdetail" label="商品详情">
					</el-table-column> -->
					<el-table-column prop="price" label="价格">
					</el-table-column>
					<el-table-column prop="createTime" label="创建时间">
						<template slot-scope="scope">
							<span>{{timestampToTime(scope.row.createTime)}}</span>
						</template>
					</el-table-column>
					<el-table-column prop="status" label="状态">
						<template slot-scope="scope">
							<span v-if="scope.row.status==0">已上架</span>
							<span v-if="scope.row.status==1">已下架</span>
						</template>
					</el-table-column>
					<el-table-column fixed="right" label="操作" width="200">
						<template slot-scope="scope">
							<el-button size="mini" @click="selectById(scope.row.gid)">编辑</el-button>
							<el-button size="mini" type="danger" @click="deleteGood(scope.row.gid)">删除</el-button>
						</template>
						<template slot="header" slot-scope="scope">
							<el-input v-model="search" size="mini" placeholder="输入关键字搜索" />
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
				<el-form-item label="商品图片">
					<input type="file" class="form-control-file" multiple="multiple" @change="changeAddImage" />
				</el-form-item>
				<el-form-item label="商品名称">
					<el-input v-model="add.gname" placeholder="请输入商品名称"></el-input>
				</el-form-item>

				<el-form-item label="商品类型名称">
					<el-select placeholder="请选择商品类型名称" v-model="add.gtypeid">
						<el-option :value="items.gtypeid" :label="items.gtypename" v-for="items in goodType" :key="items.gtypeid">
							{{items.gtypename}}
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="价格">
					<el-input v-model="add.price" placeholder="请输入商品价格" @keyup.enter="handleClick"></el-input>
				</el-form-item>

				<el-form-item label="商品状态">
					<el-select placeholder="请选择轮商品状态" v-model="add.status">
						<el-option label="上架" value="0"></el-option>
						<el-option label="下架" value="1"></el-option>
					</el-select>
				</el-form-item>

				<el-form-item label="商品详情">
					<div class="edit_container">
						<quill-editor v-model="add.gdetail" ref="addeditor" class="editer" :options="editorOption" @ready="onEditorReady($event)">
						</quill-editor>
					</div>
					<!-- <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="商品详情" v-model="add.gdetail">
					</el-input> -->
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showAddDialog = false">取 消</el-button>
				<el-button type="primary" @click="addGood()">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="修改商品类型" :visible.sync="showUpdateDialog">
			<el-form label-width="100px">
				<el-form-item label="商品图片">
					<input type="file" class="form-control-file" multiple="multiple" @change="changeUpdateImage" />
				</el-form-item>
				<el-form-item label="商品名称">
					<el-input v-model="update.gname" placeholder="请输入商品名称"></el-input>
				</el-form-item>

				<el-form-item label="商品类型名称">
					<el-select placeholder="请选择轮商品类型名称" v-model="update.gtypeid">
						<el-option :value="items.gtypeid" :label="items.gtypename" v-for="items in goodType" :key="items.gtypeid">
							{{items.gtypename}}
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="价格">
					<el-input v-model="update.price" placeholder="请输入商品价格"></el-input>
				</el-form-item>
				<el-form-item label="商品状态">
					<el-select placeholder="请选择轮商品状态" v-model="update.status">
						<el-option label="上架" :value="0"></el-option>
						<el-option label="下架" :value="1"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="商品详情">
					<div class="edit_container">
						<quill-editor v-model="update.gdetail" ref="addeditor" class="editer" :options="editorOption" @ready="onEditorReady($event)">
						</quill-editor>
					</div>
					<!-- <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="商品详情" v-model="add.gdetail">
					</el-input> -->
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showUpdateDialog = false">取 消</el-button>
				<el-button type="primary" @click="updateGood()">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	import {
		quillEditor
	} from "vue-quill-editor"; //调用编辑器

	import 'quill/dist/quill.core.css'
	import 'quill/dist/quill.snow.css'
	import 'quill/dist/quill.bubble.css'
	export default {
		name: "goodmanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				goodType: [],
				search: '',
				showAddDialog: false,
				showUpdateDialog: false,
				option: '',
				pageinfo: {},
				add: {
					gname: '',
					gtypeid: '',
					gdetail: "",
					price: "",
					createTime: "",
					status: "",
					imgFile: '',
				},
				update: {
					gid: 0,
					gname: '',
					gtypeid: '',
					gdetail: "",
					price: "",
					createTime: "",
					status: "",
					imgFile: '',
				},
				editorOption: {},
				msg: ""
			}
		},
		components: { //使用编辑器
			quillEditor
		},
		watch: {
			isClear(val) {
				// 触发清除文本域内容
				if (val) {
					this.editor.txt.clear()
				}
			},
			value(val) {
				// 使用 v-model 时，设置初始值
				this.editor.txt.html(val)
			}
		},
		mounted() {
			this.getGoodList();
		},
		methods: {
			onEditorReady(editor) {},
			/**
			 * 加载列表
			 */
			getGoodList() {
				this.axios.post("/admin/zGoods/getList", {
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
				this.axios.post("/admin/zGoods/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
			getGoodTypeList() {
				this.axios.post("/admin/goodsType/getList", {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.goodType = json.data.data;
					})
			},
			selectById(gid) {
				this.axios.post("/admin/zGoods/selectById?gid=" + gid, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.update = json.data.data;
						console.log(this.update);
						this.showUpdateDialog = true;
						this.getGoodTypeList();
					})
			},
			/**
			 * 更改保存商品类型图片框内容
			 * @param e
			 */
			changeAddImage(e) {
				console.log(e.target.files)
				this.add.imgFile = e.target.files
			},
			/**
			 * 修改模态框更换图片
			 */
			changeUpdateImage(e) {
				this.update.imgFile = e.target.files
			},
			/**
			 * 新增
			 */
			addGood() {
				let data = new FormData();
				data.append("gtypeid", this.add.gtypeid);
				data.append("gname", this.add.gname);
				data.append("gdetail", this.add.gdetail);
				data.append("price", this.add.price);
				data.append("status", this.add.status);
				// data.append("files", this.add.imgFile);

				for (let i = 0; i < this.add.imgFile.length; i++) {
					data.append('imgFile', this.add.imgFile[i])
				}
				let config = {
					//添加请求头
					headers: {
						'Content-Type': 'multipart/form-data'
					},
				};
				this.axios.post("/admin/zGoods/add", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.showAddDialog = false;
							this.msg = json.data.msg;
							this.open2();
							this.getGoodList();
							this.add = {};
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
			deleteGood(gid) {
				let flag = confirm("您确定要删除编号为[" + gid + "]的商品类型信息吗?");
				if (flag) {
					this.axios.get("/admin/zGoods/delete?gid=" + gid)
						.then((json) => {
							console.log(json)
							if (json.data.code == "200") {
								this.msg = json.data.msg;
								//消息提示
								this.open2();
								//加载列表
								this.getGoodList();
							} else {}
						})
				}
			},
			/**
			 * 修改商品类型
			 */
			updateGood() {
				let data = new FormData();
				data.append("gid", this.update.gid);
				data.append("gtypeid", this.update.gtypeid);
				data.append("gname", this.update.gname);
				data.append("gdetail", this.update.gdetail);
				data.append("price", this.update.price);
				data.append("status", this.update.status);
				if (this.update.imgFile != null && this.update.imgFile != '') {
					for (let i = 0; i < this.update.imgFile.length; i++) {
						data.append('imgFile', this.update.imgFile[i])
					}
				}
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zGoods/update", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.update = {}
							this.msg = json.data.msg;
							this.showUpdateDialog = false;
							this.open2();
							this.getGoodList();
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
			/**
			 * @param {Object} timestamp时间转换
			 */
			timestampToTime(timestamp) {
				var date = new Date(timestamp); //时间戳为10位需*1000，时间戳为13位的话不需乘1000
				var Y = date.getFullYear() + '-';
				var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';
				var D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' ';
				var h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
				var m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
				var s = (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds());
				return Y + M + D + h + m + s;
			},
			OpenAddModel() {
				this.getGoodTypeList();
				this.showAddDialog = true;
			},
		}
	}
</script>

<style scoped>

</style>
