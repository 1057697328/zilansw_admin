<template>
	<div>
		<el-row style="margin-bottom:10px;">
			<el-col :span="24">
				<div>
					<el-button type="primary" icon="el-icon-plus" @click="OpenAddModel()">添加轮播</el-button>
				</div>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<el-table :data="tableData" style="width: 100%">
					<el-table-column prop="sid" label="库存编号" width="180">
					</el-table-column>
					<el-table-column prop="zgoods.gname" label="商品名称">
					</el-table-column>
					<el-table-column prop="stocknum" label="库存数量">
					</el-table-column>
					<el-table-column fixed="right" label="操作" width="200">
						<template slot-scope="scope">
							<el-button size="mini" @click="getData(scope.row)">编辑</el-button>
							<el-button size="mini" type="danger" @click="deleteStock(scope.row.sid)">删除</el-button>
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
				<el-form-item label="商品名称">
					<el-select placeholder="请输入商品名称" v-model="add.gid">
						<el-option :label="items.gname" :value="items.gid" v-for="items in goodData" :key="items.gid"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="库存数量">
					<el-input v-model="add.stocknum" placeholder="请输入库存数量"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showAddDialog = false">取 消</el-button>
				<el-button type="primary" @click="addStock()">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="修改轮播" :visible.sync="showUpdateDialog">
			<el-form label-width="100px">
				<el-form-item label="商品名称">
					<el-input v-model="update.zgoods.gname" :disabled="true" placeholder="请输入商品名称"></el-input>
				</el-form-item>
				<el-form-item label="库存数量">
					<el-input v-model="update.stocknum" placeholder="请输入库存数量"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="showUpdateDialog = false">取 消</el-button>
				<el-button type="primary" @click="updateStock()">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>


<script>
	import CONSTANT from '@/assets/constant'
	export default {
		name: "stockmanager",
		data() {
			return {
				CONSTANT: CONSTANT,
				tableData: [],
				goodData: [],
				showAddDialog: false,
				showUpdateDialog: false,
				option: '',
				pageinfo: {},
				add: {
					gid: "",
					stocknum: ""
				},
				update: {
					sid: 0,
					gid: "",
					stocknum: "",
					zgoods: []
				},
				dialogImageUrl: '',
				msg: ""
			}
		},
		mounted() {
			this.getStockList();
		},
		methods: {
			/**
			 * 加载列表
			 */
			getStockList() {
				this.axios.post("/admin/zStock/getList", {
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
				this.axios.post("/admin/zStock/getList?pageIndex=" + pageIndex, {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.tableData = json.data.data;
						this.pageinfo = json.data.pageBean;
					})
			},
			/**
			 * 查询所有商品
			 */
			getGoodList() {
				this.axios.post("/admin/zGoods/selectAll", {
						headers: {
							'Content-Type': 'application/x-www-form-urlencoded'
						},
					})
					.then((json) => {
						this.goodData = json.data.data
					})
			},
			/**
			 * @param {Object} sid根据编号查询
			 */
			getData(obj) {
				this.update = obj;
				this.showUpdateDialog = true;
			},
			/**
			 * 新增
			 */
			addStock() {
				let data = new FormData();
				data.append("gid", this.add.gid);
				data.append("stocknum", this.add.stocknum);
				this.axios.post("/admin/zStock/add", data)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							this.showAddDialog = false;
							this.msg = json.data.msg;
							this.open2();
							this.getStockList();
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
			deleteStock(sid) {
				let flag = confirm("您确定要删除编号为[" + sid + "]的轮播信息吗?");
				if (flag) {
					this.axios.get("/admin/zStock/delete?sid=" + sid)
						.then((json) => {
							console.log(json)
							if (json.data.code == "200") {
								this.msg = json.data.msg;
								//消息提示
								this.open2();
								//加载列表
								this.getStockList();
								this.getGoodList();
							} else {}
						})
				}
			},
			/**
			 * 修改库存数量
			 */
			updateStock() {
				let data = new FormData();
				data.append("sid", this.update.sid);
				data.append("gid", this.update.gid);
				data.append("stocknum", this.update.stocknum);
				this.axios.post("/admin/zStock/update", data)
					.then((json) => {
						if (json.data.code === CONSTANT.statusCode.SUCCESS) {
							// this.update = {}
							this.msg = json.data.msg;
							this.showUpdateDialog = false;
							this.open2();
							this.getStockList();
							this.getGoodList();
						} else {
							CONSTANT.MESSAGEBOX(json.data.message, "success", () => {})
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
			 * 打开新增模态框
			 */
			OpenAddModel() {
				this.getGoodList();
				this.showAddDialog = true;
			},
		}
	}
</script>

<style scoped>

</style>
