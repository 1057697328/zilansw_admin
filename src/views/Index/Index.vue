<!--suppress ALL -->
<template>
	<div>
		<el-container>
			<el-header style="background-color: rgb(238, 241, 246)" :height="'80px'">
				<h1 style="padding:1rem 0 0 0;color:gray">湖南紫澜生物科技有限公司</h1>
				<h4>微信公众号管理系统</h4>
			</el-header>
			<el-container>
				<el-aside style="background-color: rgb(238, 241, 246)">
					<el-menu :default-openeds="['1']">
						<el-submenu index="1">
							<template slot="title"><i class="el-icon-setting"></i>后台管理</template>
							<el-menu-item-group title="轮播">
								<el-menu-item index="1-1" @click="chooseComponent('bannermanager')"><i class="el-icon-picture-outline"></i>首页轮播管理</el-menu-item>
							</el-menu-item-group>
							<el-menu-item-group title="商品">

								<el-menu-item index="1-2" @click="chooseComponent('goodtypemanager')"><i class="el-icon-collection"></i>商品类型管理</el-menu-item>
								<el-menu-item index="1-3" @click="chooseComponent('goodmanager')"><i class="el-icon-goods"></i>商品管理</el-menu-item>
								<el-menu-item index="1-4" @click="chooseComponent('stockmanager')"><i class=el-icon-notebook-1></i>库存管理</el-menu-item>
							</el-menu-item-group>
							<el-menu-item-group title="订单">
								<el-menu-item index="3-1"><i class=el-icon-tickets></i>订单管理</el-menu-item>
							</el-menu-item-group>
							<el-menu-item-group title="其它">
								<el-menu-item index="4-1" @click="chooseComponent('messagemanager')"><i class=el-icon-folder-opened></i>商城公告管理</el-menu-item>
								<el-menu-item index="4-2"><i class="el-icon-user"></i>用户管理</el-menu-item>
								<el-menu-item index="4-3" @click="chooseComponent('evaluationmanager')"><i class="el-icon-star-off"></i>商品评价管理</el-menu-item>
								<el-menu-item index="4-4" @click="chooseComponent('adminmanager')"><i class="el-icon-user"></i>管理员管理</el-menu-item>
							</el-menu-item-group>
						</el-submenu>
					</el-menu>
				</el-aside>
				<el-main>
					<bannermanager v-if="componentName==='bannermanager'"></bannermanager>
					<goodtypemanager v-if="componentName==='goodtypemanager'"></goodtypemanager>
					<goodmanager v-if="componentName==='goodmanager'"></goodmanager>
					<stockmanager v-if="componentName==='stockmanager'"></stockmanager>
					<messagemanager v-if="componentName==='messagemanager'"></messagemanager>
					<evaluationmanager v-if="componentName==='evaluationmanager'"></evaluationmanager>
					<adminmanager v-if="componentName==='adminmanager'"></adminmanager>

				</el-main>
			</el-container>
		</el-container>
	</div>
</template>

<script>
	/**
	 * 轮播管理
	 */
	import BannerManager from '@/components/banner/bannermanager'
	import Bannermanager from "../../components/banner/bannermanager";

	/**
	 * 商品类型管理
	 */
	import GoodTypeManager from '@/components/goodType/goodtypemanager'
	import Goodtypemanager from "../../components/goodType/goodtypemanager";

	/**
	 * 商品管理
	 */
	import GoodManager from '@/components/good/goodmanager'
	import Goodmanager from "../../components/good/goodmanager";

	/**
	 * 库存管理
	 */
	import StockManager from '@/components/stock/stockmanager'
	import Stockmanager from "../../components/stock/stockmanager";

	/**
	 * 消息公告管理
	 */
	import MessageManager from '@/components/message/messagemanager'
	import Messagemanager from "../../components/message/messagemanager";

	/**
	 * 评论管理
	 */
	import EvaluationManager from '@/components/evaluation/evaluationmanager'
	import Evaluationmanager from "../../components/evaluation/evaluationmanager";

	/**
	 * 管理员管理
	 */
	import AdminManager from '@/components/admin/adminmanager'
	import Adminmanager from "../../components/admin/adminmanager";
	export default {
		name: "Index",
		components: {
			Bannermanager,
			Goodtypemanager,
			Goodmanager,
			Stockmanager,
			Messagemanager,
			Evaluationmanager,
			Adminmanager,
		},
		data() {
			return {
				componentName: ''
			}
		},
		methods: {
			/**
			 * 展示组件
			 * @param name 组件名称
			 */
			chooseComponent(name) {
				this.componentName = name;
			}
		}
	}
</script>

<style scoped>

</style>
