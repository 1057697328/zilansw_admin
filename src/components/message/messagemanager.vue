<template>
	<div>
		<el-row style="margin-bottom:10px;">
			<el-col :span="24">
				<div>
					<el-button type="primary" icon="el-icon-plus" @click="showAddDialog=true">添加消息公告</el-button>
				</div>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData" style="width: 100%">
					<el-table-column prop="mid" label="公告编号" width="180">
					</el-table-column>
					<el-table-column prop="title" label="公告标题">
					</el-table-column>
					<!-- <el-table-column prop="content" label="公告正文">
					</el-table-column> -->
					<el-table-column prop="author" label="作者">
					</el-table-column>
					<el-table-column prop="createTime" label="公告创建时间">
						<template slot-scope="scope">
							<span>{{timestampToTime(scope.row.createTime)}}</span>
						</template>
					</el-table-column>
					<el-table-column fixed="right" label="操作" width="200">
						<template slot-scope="scope">
							<el-button size="mini" @click="getData(scope.row)">编辑</el-button>
							<el-button size="mini" type="danger" @click="deleteMessage(scope.row.mid)">删除</el-button>
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

		<el-dialog title="添加消息公告" :visible.sync="showAddDialog">
			<el-form label-width="100px">
				<el-form-item label="公告标题">
					<el-input v-model="add.title" placeholder="请输入公告标题"></el-input>
				</el-form-item>
				<el-form-item label="作者">
					<el-input v-model="add.author" placeholder="请输入作者"></el-input>
				</el-form-item>
				<el-form-item label="公告正文" prop="words">
					<div class="edit_container">
						<quill-editor v-model="add.content" ref="addeditor" class="editer" :options="editorOption" @ready="onEditorReady($event)">
						</quill-editor>
					</div>
				</el-form-item>
				<!-- <el-form-item label="公告正文">
					<div ref="addeditor" id="editorElem" style="text-align:left"></div>
				</el-form-item> -->
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showAddDialog = false">取 消</el-button>
				<el-button type="primary" @click="addMessage()">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="修改消息公告" :visible.sync="showUpdateDialog">
			<el-form label-width="100px">
				<el-form-item label="公告标题">
					<el-input v-model="update.title" placeholder="请输入公告标题"></el-input>
				</el-form-item>
				<el-form-item label="作者">
					<el-input v-model="update.author" placeholder="请输入作者"></el-input>
				</el-form-item>
				<el-form-item label="公告正文" prop="words">
					<div class="edit_container">
						<quill-editor v-model="update.content" ref="updateeditor" class="editer" :options="editorOption" @ready="onEditorReady($event)">
						</quill-editor>
					</div>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showUpdateDialog = false">取 消</el-button>
				<el-button type="primary" @click="updateMessage()">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	// import E from 'wangeditor'
	import {
		quillEditor
	} from "vue-quill-editor"; //调用编辑器

	import 'quill/dist/quill.core.css'
	import 'quill/dist/quill.snow.css'
	import 'quill/dist/quill.bubble.css'
	export default {
		name: "messagemanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				showAddDialog: false,
				showUpdateDialog: false,
				option: '',
				pageinfo: {},
				editorOption: {},
				add: {
					createTime: '',
					title: '',
					content: "",
					author: ''
				},
				update: {
					mid: 0,
					createTime: '',
					title: '',
					content: "",
					author: ''
				},
				dialogImageUrl: '',
				msg: "",
				addeditor: null,
				updateeditor: null,
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
			this.getMessageList();
		},
		methods: {
			onEditorReady(editor){
			},
			/**
			 * 加载列表
			 */
			getMessageList() {
				this.axios.post("/admin/zmessage/getList", {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data
						// this.updateeditor.txt.html(this.update.content)
						this.pageinfo = json.data.pageBean;
					})
			},
			/**
			 * @param {Object} pageIndex获取分页数据
			 */
			getPage(pageIndex) {
				this.axios.post("/admin/zmessage/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
			getData(obj) {
				this.update = obj;
				this.showUpdateDialog = true;
			},
			/**
			 * 新增
			 */
			addMessage() {
				let data = new FormData();
				data.append("title", this.add.title);
				data.append("content", this.add.content);
				data.append("author", this.add.author);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zmessage/add", data,config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.showAddDialog = false;
							this.msg = json.data.msg;
							this.open2();
							this.getMessageList();

						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {
								this.showAddDialog = false;
							})
						}
					})
			},
			/**
			 * 删除消息公告
			 * @param bid
			 */
			deleteMessage(mid) {
				let flag = confirm("您确定要删除编号为[" + mid + "]的消息公告信息吗?");
				if (flag) {
					this.axios.get("/admin/zmessage/delete?mid=" + mid)
						.then((json) => {
							console.log(json)
							if (json.data.code == "200") {
								this.msg = json.data.msg;
								//消息提示
								this.open2();
								//加载列表
								this.getMessageList();
							} else {}
						})
				}
			},
			/**
			 * 修改消息公告
			 */
			updateMessage() {
				let data = new FormData();
				data.append("mid", this.update.mid);
				data.append("title", this.update.title);
				data.append("content", this.update.content);
				data.append("author", this.update.author);
				let config = {
					//添加请求头
					headers: {
						"Content-Type": "multipart/form-data"
					},
				};
				this.axios.post("/admin/zmessage/update", data,config)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.update = {}
							this.msg = json.data.msg;
							this.showUpdateDialog = false;
							this.open2();
							this.getMessageList();
						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {
								jQuery('#updateModal').modal('hide');
							})
						}
					})
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
