<style lang="less">
	@import 'dataSourceIndex.less';
</style>
<template>
	<div class="dataSource">
		<!-- 筛选 -->
		<Form inline :model="formItem" style="position:relative;width:100%;margin: 15px 0 0 0;">
			<FormItem>
				<span style="margin-left:15px;">检索内容 ：</span>
			</FormItem>
			<FormItem>
				<AutoComplete v-model="formItem.serchName" :clearable="clearable" icon="ios-search" placeholder="请输入要检索的日志名称或日志标识"
				 style="width: 325px;" @on-change="searchName(formItem.serchName)"></AutoComplete>
			</FormItem>
		</Form>
		<Table :loading="loading" :highlight-row="highlight" @on-row-click="showTableData" stripe :columns="tableColumns"
		 :data="tableData" :height="tableHeight"></Table>
		<Page :total="totalItem" :page-size="pageSize" size="small" show-elevator show-total style="margin:15px 0 0 0;text-align: center;font-size: 14px;"
		 show-sizer :page-size-opts="sizeOpts" @on-change="changeNo" @on-page-size-change="changeSize"></Page>
		<!-- 抽屉-->
		<Drawer :title="drawerData.LOGINFO_TYPE" :closable="false" placement="left" width="650" v-model="drawerLeft">
			<Form :model="drawerData" :label-width="80">
				<FormItem label="日志标识"></b>
					<b style="font-size: 18px;">{{drawerData.LOGINFO_TYPE}}</b>
				</FormItem>
				<FormItem label="日志内容">
					<b style="font-size: 18px;">{{drawerData.LOGINFO_LOGS}}</b>
				</FormItem>
				<FormItem label="记录人">
					<b style="font-size: 18px;">{{drawerData.LOGINFO_LOGS}}</b>
				</FormItem>
				<FormItem label="记录时间">
					<b style="font-size: 18px;">{{drawerData.LOGINFO_LOGS}}</b>
				</FormItem>
			</Form>
		</Drawer>
		<Modal v-model="modalType" width="500px" :mask-closable="false">
			<p slot="header" style="color:#f60;text-align:center">
				<Icon type="ios-information-circle"></Icon>
				<span>操作确认</span>
			</p>
			<div style="text-align:center">
				<p style="font-size: 16px;">{{modelTap.split(",")[0]}}</p>
				<br />
				<p style="font-size: 16px;">{{modelTap.split(",")[1]}}</p>
			</div>
			<div slot="footer">
				<Button type="error" size="large" long @click="remove">确认删除</Button>
			</div>
		</Modal>
	</div>
</template>
<script>
	import notices from '../../notice.js'
	import moment from 'moment';
	export default {
		data() {
			return {
				//模态框
				modalType: false,
				modelTap: '',
				modelId: '', //待确认删除的id
				modelVersion: '', //待确认删除的数据version
				//抽屉
				drawerLeft: false,
				drawerRight: false,
				drawerData:{},
				isEdit: false,
				styles: {
					height: 'calc(100% - 55px)',
					overflow: 'auto',
					paddingBottom: '53px',
					position: 'static'
				},
				//筛选框
				clearable: true,
				formItem: {
					serchName: ''
				},
				tableHeight: '490',
				//Table数据相关
				loading: true,
				highlight: true,
				totalItem: 0, // 数据总数
				pageSize: 10, // 默认10条
				pageNo: 1, // 默认第一页
				tableData: [],
				sizeOpts: [5, 10, 15, 30],
				tableColumns: [
					{
						title: '日志标识',
						key: 'LOGINFO_TYPE',
						render: (h, params) => {
							return h('div', [
								h('span', params.row.LOGINFO_TYPE == "0" ? "系统登录" : params.row.LOGINFO_TYPE == "1" ? "数据录入" :  params.row.LOGINFO_TYPE == "2" ? "数据更新" : "数据删除")
							]);
						}
					},
					{
						title: '日志内容',
						key: 'LOGINFO_LOGS'
					},{
						title: '日志记录人',
						key: 'SYSUSER1_NAME'
					},
					{
						title: '记录时间',
						key: 'LOGINFO_CREATETIME',
						render: (h, params) => {
							return h('div', [
								h('span', moment(params.row.DTINFO_CREATETIME).format("YYYY-MM-DD HH:mm:ss"))
							]);
						}
					},{
						title: '操作',
						key: 'action',
						width: 250,
						align: 'center',
						render: (h, params) => {
							return h('div', [
								h('Button', {
									props: {
										type: 'error',
										size: 'small'
									},
									on: {
										click: () => {
											this.isEdit = true
											this.modalType = true
											this.modelVersion = params.row.LOGINFO_VERSION + 1
											this.modelId = params.row.LOGINFO_ID
											this.modelTap = '您将删除：' + params.row.LOGINFO_NAME + ",数据一旦删除将无法恢复，是否确认无误?"
										}
									}
								}, '删除')
							]);
						}
					}
				]
			}
		},
		methods: {
			//删除方法
			remove() {
				var _this = this;
				this.$http.deleteForce(this,
					"LOGINFO",
					this.modelId,
					0, res => {
						if (res.data.search("成功") != -1) {
							//数据参数重置
							this.isEdit = false
							this.modalType = false
							this.getTableData()
							notices.notice(_this, '操作提示', '日志已删除成功。', 1)
						} else {
							notices.notice(_this, '操作提示', res.data == '' ? '意料之外的错误,请联系管理员' : res.data, 3)
						}
					})
			},
			//关闭抽屉并重置相关属性参数
			cancel() {
				this.isEdit = false
				this.drawerRight = false
				this.getTableData()
			},
			//打开查询的抽屉
			showTableData(data, index) {
				if (!this.isEdit) {
					this.drawerLeft = true
					this.drawerData = data
				}
			},
			//获取Table表单数据
			getTableData() {
				var _this = this;
				_this.loading = true;
				let params = {
					'LOGINFO_LOGS#LIKE': this.formItem.serchName == '' ? null : '%' + this.formItem.serchName + '%',
					'W!LOGINFO_ISDELETE#EQ': 0,
				}

				this.$http.selectBase(this,
					"LOGINFO,SYSUSER1",
					"LOGINFO",
					"SYSUSER1_NAME",
					"LOGINFO_CREATETIME",
					"4",
					_this.pageNo, _this.pageSize,
					params, 0, res => {
						_this.tableData = res.data.data;
						_this.totalItem = res.data.total;
						_this.loading = false;
					})
			},
			//分页方法
			changeNo(n) {
				this.pageNo = n;
				this.getTableData();
			},
			//改变每页显示条数
			changeSize(n) {
				this.pageSize = n;
				this.getTableData();
			},
			//检索方法
			searchName(name) {
				this.formItem = {
					serchName: name
				}
				this.getTableData();
			},
			//设置Table的高度
			setTableHeight() {
				this.tableHeight = ($(window).height() - 233);
			}
		},
		created() {
			this.getTableData();
			this.setTableHeight();
		},
		//启用监听,以监听当前页面高度控制Table的Height值
		mounted() {
			const that = this
			window.onresize = () => {
				return (() => {
					this.setTableHeight();
				})()
			}
		},
		watch: {
			tableHeight(val) {
				if (!this.timer) {
					this.timer = true
					let that = this
					//以定时器优化监听的频繁调用造成的页面卡顿
					setTimeout(function() {
						that.timer = false
					}, 500)
				}
			}
		}
	}
</script>
<style>
	/*抽屉样式*/
	.demo-drawer-footer {
		width: 100%;
		position: absolute;
		bottom: 0;
		left: 0;
		border-top: 1px solid #e8e8e8;
		padding: 10px 16px;
		text-align: right;
		background: #fff;
	}

	.vertical-center-modal {
		display: flex;
		align-items: center;
		justify-content: center;

		.ivu-modal {
			top: 0;
		}
	}
</style>
