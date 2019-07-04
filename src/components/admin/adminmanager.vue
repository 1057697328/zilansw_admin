<template>
	<div>
		<el-row style="margin-bottom:10px;">
			<el-col :span="24">
				<div>
					<el-button type="primary" icon="el-icon-plus" @click="showAddDialog=true">添加管理员</el-button>
				</div>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData" style="width: 100%">
					<el-table-column prop="aid" label="管理员编号" width="180">
					</el-table-column>
					<el-table-column prop="account" label="账号">
					</el-table-column>
					<el-table-column prop="name" label="姓名">
					</el-table-column>
					<el-table-column prop="status" label="状态">
						<template slot-scope="scope">
							<span v-if="scope.row.status==0">正常</span>
							<span v-if="scope.row.status==1">禁用</span>
						</template>
					</el-table-column>
					<el-table-column fixed="right" label="操作" width="200">
						<template slot-scope="scope">
							<el-button size="mini" @click="getDate(scope.row)">编辑</el-button>
							<el-button size="mini" type="danger" @click="deleteAdmin(scope.row.aid)">删除</el-button>
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

		<el-dialog title="添加管理员" :visible.sync="showAddDialog">
			<el-form label-width="100px">
				<el-form-item label="账号">
					<el-input v-model="add.account" placeholder="请输入账号"></el-input>
				</el-form-item>
				<el-form-item label="密码">
					<el-input v-model="add.password" placeholder="请输入密码"></el-input>
				</el-form-item>
				<el-form-item label="姓名">
					<el-input v-model="add.name" placeholder="请输入姓名"></el-input>
				</el-form-item>
				<el-form-item label="管理员状态">
					<el-select placeholder="请选择管理员状态" v-model="add.status">
						<el-option label="正常" value="0"></el-option>
						<el-option label="禁用" value="1"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showAddDialog = false">取 消</el-button>
				<el-button type="primary" @click="addAdmin()">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="修改管理员" :visible.sync="showUpdateDialog">
			<el-form label-width="100px">
				<el-form-item label="账号">
					<el-input v-model="update.account" placeholder="请输入账号"></el-input>
				</el-form-item>
				<el-form-item label="密码">
					<el-input v-model="update.password" placeholder="请输入密码"></el-input>
				</el-form-item>
				<el-form-item label="姓名">
					<el-input v-model="update.name" placeholder="请输入姓名"></el-input>
				</el-form-item>
				<el-form-item label="管理员状态">
					<el-select placeholder="请选择管理员状态" v-model="update.status">
						<el-option label="正常" :value="0"></el-option>
						<el-option label="禁用" :value="1"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showUpdateDialog = false">取 消</el-button>
				<el-button type="primary" @click="updateAdmin()">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	export default {
		name: "adminmanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				showAddDialog: false,
				showUpdateDialog: false,
				option: '',
				pageinfo: {},
				add: {
					account: '',
					password: '',
					status: '',
					name: ''
				},
				update: {
					aid: 0,
					account: '',
					password: '',
					status: '',
					name: ''
				},
				dialogImageUrl: '',
				msg: ""
			}
		},
		mounted() {
			this.getAdminList();
		},
		methods: {
			/**
			 * 加载列表
			 */
			getAdminList() {
				this.axios.post("/admin/zAdmin/getList", {
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
				this.axios.post("/admin/zAdmin/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
			getDate(obj) {
				this.update = obj;
				this.showUpdateDialog = true;
			},
			/**
			 * 新增
			 */
			addAdmin() {
				let data = new FormData();
				data.append("account", this.add.account);
				data.append("password", this.add.password);
				data.append("name", this.add.name);
				data.append("status", this.add.status);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zAdmin/add", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.showAddDialog = false;
							this.msg = json.data.msg;
							this.open2();
							this.getAdminList();

						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {
								this.showAddDialog = false;
							})
						}
					})
			},
			/**
			 * 删除管理员
			 * @param aid
			 */
			deleteAdmin(aid) {
				let flag = confirm("您确定要删除编号为[" + aid + "]的管理员信息吗?");
				if (flag) {
					this.axios.get("/admin/zAdmin/delete?aid=" + aid)
						.then((json) => {
							console.log(json)
							if (json.data.code == "200") {
								this.msg = json.data.msg;
								//消息提示
								this.open2();
								//加载列表
								this.getAdminList();
							}
							if (json.data.code === CONSTANT.statusCode.JURISFICTION) {
								this.msg = json.data.msg;
								this.open2("error");
								this.getAdminList();
							}
						})
				}
			},
			/**
			 * 修改管理员
			 */
			updateAdmin() {
				let data = new FormData();
				data.append("aid", this.update.aid);
				data.append("account", this.update.account);
				data.append("password", this.update.password);
				data.append("name", this.update.name);
				data.append("status", this.update.status);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zAdmin/update", data, config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.update = {}
							this.msg = json.data.msg;
							this.showUpdateDialog = false;
							this.open2();
							this.getAdminList();
						}

					})
			},
			/**
			 * 消息提示
			 */
			open2(type) {
				if (type != null && type != "" && type != undefined) {
					type = type
				} else {
					type = 'success'
				}
				this.$message({
					message: this.msg,
					type: type

				});
			},
		}
	}
</script>

<style scoped>

</style>
