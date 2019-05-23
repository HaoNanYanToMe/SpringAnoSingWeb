<style lang="less">
	@import '../roles/dataSourceIndex.less';
</style>
<template>
	<div class="dataSource">
		<!-- 筛选 -->
		<Form inline :model="formItem" style="position:relative;width:100%;margin: 15px 0 0 0;">
			<FormItem>
				<span style="margin-left:15px;">检索内容 ：</span>
			</FormItem>
			<FormItem>
				<AutoComplete v-model="formItem.serchName" :clearable="clearable" icon="ios-search" placeholder="请输入要检索的药品名称或药品标识"
				 style="width: 325px;" @on-change="searchName(formItem.serchName)"></AutoComplete>
			</FormItem>
			<FormItem style="width: 175px;">
				<Select v-model="formItem.uid" @on-change="getUid" placeholder="请选择数据范围">
					<Option value="0" key="0">全部</Option>
					<Option value="1" key="1">仅自己</Option>
				</Select>
			</FormItem>
			<FormItem style="width: 175px;">
				<Select v-model="formItem.isDelete" @on-change="getDelete" placeholder="请选择数据范围">
					<Option value="-2" key="-2">待审核</Option>
					<Option value="-1" key="-1">全部</Option>
					<Option value="0" key="0">正常</Option>
					<Option value="1" key="1">被拒绝</Option>
				</Select>
			</FormItem>
			</FormItem>
			<FormItem style="float:right;">
				<Button type="primary" @click="add">新增药品信息</Button>
			</FormItem>
		</Form>
		<Table :loading="loading" :highlight-row="highlight" @on-row-click="showTableData" stripe :columns="tableColumns"
		 :data="tableData" :height="tableHeight"></Table>
		<Page :total="totalItem" :page-size="pageSize" size="small" show-elevator show-total style="margin:15px 0 0 0;text-align: center;font-size: 14px;"
		 show-sizer :page-size-opts="sizeOpts" @on-change="changeNo" @on-page-size-change="changeSize"></Page>
		<!-- 抽屉-->
		<Drawer :title="drawerData.DRUGINFO_NAME" :closable="false" placement="left" width="650" v-model="drawerLeft">
			<Form :model="drawerData" :label-width="80">
				<FormItem label="药品名称"></b>
					<b style="font-size: 18px;">{{drawerData.DRUGINFO_NAME}}</b>
				</FormItem>
				<FormItem label="供应商">
					<b style="font-size: 18px;">{{drawerData.DRUGINFO_SUPPLIER}}</b>
				</FormItem>
			</Form>
		</Drawer>
		<Drawer :title="drawerTap" placement="right" width="650" v-model="drawerRight" :mask-closable="false" @on-close="cancel"
		 :styles="styles">
			<Form :model="drawerData" :label-width="80">
				<FormItem label="药品名称">
					<Input v-model="drawerData.DRUGINFO_NAME" placeholder="请输入药品表名称,如:阿司匹林"></Input>
				</FormItem>
				<FormItem label="供应商名称">
					<Input v-model="drawerData.DRUGINFO_SUPPLIER" placeholder="请输入供应商名称,如:祥和制药"></Input>
				</FormItem>
				<FormItem label="供应商电话">
					<Input v-model="drawerData.DRUGINFO_PHONE" placeholder="请输入供应商电话,如:123456"></Input>
				</FormItem>
				<FormItem label="库存数量">
					<Input v-model="drawerData.DRUGINFO_STOCK" placeholder="请输入药品表库存数量,如:0"></Input>
				</FormItem>
				<FormItem label="库存数量">
					<Select v-model="drawerData.DRUGINFO_TID" @on-change="getType" style="width:220px"
						placeholder="请选择分类">
						<Option value="-1" key="-1">全部</Option>
						<Option v-for="item in typeList" :value="item.DTINFO_ID" :key="item.DTINFO_ID">{{ item.DTINFO_NAME }}</Option>
					</Select>
				</FormItem>
			</Form>
			<div class="demo-drawer-footer">
				<Button style="margin-right: 8px" @click="cancel" ref='cancel'>取消操作</Button>
				<Button type="primary" @click="drawerSubmit">确认操作</Button>
			</div>
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
		<Modal v-model="modalType2" width="500px" :mask-closable="false">
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
				<Button type="info" size="large" long @click="drawerSubmit">确认审核通过</Button>
			</div>
		</Modal>
		<Modal v-model="modalType3" width="500px" :mask-closable="false">
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
				<Button type="error" size="large" long @click="drawerSubmit">确认驳回审核</Button>
			</div>
		</Modal>
		<Modal v-model="modalType4" width="500px" :mask-closable="false">
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
				<Button type="info" size="large" long @click="drawerSubmit">确认提交审核</Button>
			</div>
		</Modal>
	</div>
</template>
<script>
	import notices from '../../notice.js'
	import Cookies from 'js-cookie';
	import moment from 'moment';

	export default {
		data() {
			return {
				typeList:[],
				//数据状态
				//-2:新增时  0：点击审核通过后  1：点击驳回后
				editType: "-2",
				//模态框
				modalType: false,
				modalType2: false,
				modalType3: false,
				modalType4: false,
				modelTap: '',
				modelId: '', //待确认删除的id
				modelVersion: '', //待确认删除的数据version
				//抽屉
				drawerLeft: false,
				drawerRight: false,
				drawerData: {},
				drawerTap: '编辑',
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
					serchName: '',
					isDelete: '-1',
					uid: '0',
				},
				tableHeight: '490',
				//Table数据相关
				loading: true,
				highlight: true,
				totalItem: 0, // 数据总数
				pageSize: 10, // 默认10条
				pageNo: 1, // 默认第一页
				tableData: [],
				sizeOpts: [15, 20, 25, 30],
				tableColumns: [{
						title: '药品名称',
						key: 'DRUGINFO_NAME'
					},
					{
						title: '药品库存',
						key: 'DRUGINFO_STOCK'
					},
					{
						title: '供应商',
						key: 'DRUGINFO_SUPPLIER'
					},
					{
						title: '供应商电话',
						key: 'DRUGINFO_PHONE'
					},
					{
						title: '药品分类',
						key: 'DTINFO1_NAME'
					},
					{
						title: '创建人',
						key: 'SYSUSER1_NAME'
					},
					{
						title: '创建时间',
						key: 'DRUGINFO_CREATETIME',
						render: (h, params) => {
							return h('div', [
								h('span', moment(params.row.DRUGINFO_CREATETIME).format("YYYY-MM-DD HH:mm:ss"))
							]);
						}
					}, {
						title: '状态',
						key: 'DRUGINFO_ISDELETE',
						render: (h, params) => {
							return h('div', [
								h('span', params.row.DRUGINFO_ISDELETE == "0" ? "正常" : params.row.DRUGINFO_ISDELETE == "-2" ? "待审核" : "被驳回")
							]);
						}
					}, {
						title: '操作',
						key: 'action',
						width: 500,
						align: 'center',
						render: (h, params) => {
							return h('div', [
								h('Button', {
									props: {
										type: 'info',
										size: 'small'
									},
									style: {
										marginRight: '5px'
									},
									on: {
										click: () => {
											this.edit(params, params.index)
										}
									}
								}, '编辑'),
								h('Button', {
									props: {
										type: 'error',
										size: 'small'
									},
									style: {
										marginRight: '5px',
									},
									on: {
										click: () => {
											this.isEdit = true
											this.modalType = true
											this.modelVersion = params.row.DRUGINFO_VERSION + 1
											this.modelId = params.row.DRUGINFO_ID
											this.modelTap = '您将删除：' + params.row.DRUGINFO_NAME + ",数据一旦删除将无法恢复，是否确认无误?"
										}
									}
								}, '删除'),
								h('Button', {
									props: {
										type: 'success',
										size: 'small'
									},
									style: {
										marginRight: '5px',
										display: Cookies.get("singer") == 0 && params.row.DRUGINFO_ISDELETE == -2 ? "inline-table" : "none"
									},
									on: {
										click: () => {
											this.isEdit = true
											this.modalType2 = true
											this.modelVersion = params.row.DRUGINFO_VERSION + 1
											this.modelId = params.row.DRUGINFO_ID
											this.editType = 0
											this.drawerData = params.row
											this.modelTap = '审核：' + params.row.DRUGINFO_NAME + ",是否确认无误并审核通过?"
										}
									}
								}, '审核通过'),
								h('Button', {
									props: {
										type: 'error',
										size: 'small'
									},
									style: {
										marginRight: '5px',
										display: Cookies.get("singer") == 0 && params.row.DRUGINFO_ISDELETE == -2 ? "inline-table" : "none"
									},
									on: {
										click: () => {
											this.isEdit = true
											this.modalType3 = true
											this.modelVersion = params.row.DRUGINFO_VERSION + 1
											this.modelId = params.row.DRUGINFO_ID
											this.editType = 1
											this.drawerData = params.row
											this.modelTap = '审核：' + params.row.DRUGINFO_NAME + ",是否确认驳回该申请?"
										}
									}
								}, '审核驳回'),
								h('Button', {
									props: {
										type: 'info',
										size: 'small'
									},
									style: {
										marginRight: '5px',
										display: params.row.DRUGINFO_ISDELETE == 1 ? "inline-table" : "none"
									},
									on: {
										click: () => {
											this.isEdit = true
											this.modalType3 = true
											this.modelVersion = params.row.DRUGINFO_VERSION + 1
											this.modelId = params.row.DRUGINFO_ID
											this.editType = -2
											this.drawerData = params.row
											this.modelTap = '审核：' + params.row.DRUGINFO_NAME + ",是否重新递交该申请?"
										}
									}
								}, '重新提审')
							]);
						}
					}
				]
			}
		},
		methods: {
			getType(val){
				this.drawerData.DRUGINFO_TID = val;
			},
			//删除方法
			remove() {
				var _this = this;
				this.$http.deleteForce(this,
					"DRUGINFO",
					this.modelId,
					0, res => {
						if (res.data.search("成功") != -1) {
							//数据参数重置
							this.isEdit = false
							this.modalType = false
							this.getTableData()
							notices.notice(_this, '操作提示', '药品已删除成功。', 1)
							this.log(this.drawerData.DRUGINFO_NAME,3,1);
						} else {
							notices.notice(_this, '操作提示', res.data == '' ? '意料之外的错误,请联系管理员' : res.data, 3)
							this.log(this.drawerData.DRUGINFO_NAME,3,0);
						}
					})
			},
			//关闭抽屉并重置相关属性参数
			cancel() {
				this.isEdit = false
				this.drawerRight = false
				this.modalType1 = false
				this.modalType2 = false
				this.modalType3 = false
				this.getTableData()
			},
			getDelete(val) {
				this.formItem.isDelete = val;
				this.getTableData();
			},
			getUid(val) {
				this.formItem.uid = val;
				this.getTableData();
			},
			//信息保存
			drawerSubmit() {
				var _this = this;

				if (this.drawerData.DRUGINFO_ID) {
					//修改
					var params = {
						"DRUGINFO_NAME": this.drawerData.DRUGINFO_NAME,
						"DRUGINFO_STOCK": this.drawerData.DRUGINFO_STOCK,
						"DRUGINFO_SUPPLIER": this.drawerData.DRUGINFO_SUPPLIER,
						"DRUGINFO_UID": Cookies.get("UUID"),
						"DRUGINFO_TID": this.drawerData.DRUGINFO_TID,
						"DRUGINFO_ISDELETE": this.editType,
						"DRUGINFO_PHONE":this.drawerData.DRUGINFO_PHONE,
					}
					this.$http.updateData(this,
						"DRUGINFO",
						params,
						this.drawerData.DRUGINFO_VERSION + 1,
						this.drawerData.DRUGINFO_ID,
						0, res => {
							if (res.data.search("成功") != -1) {
								notices.notice(_this, '操作提示', '药品:' + this.drawerData.DRUGINFO_NAME + '更新成功。', 1)
								this.log(this.drawerData.DRUGINFO_NAME,2,1);
								//调用关闭，更新状态参数
								this.cancel();
							} else {
								notices.notice(_this, '操作提示', res.data == '' ? '意料之外的错误,请联系管理员' : res.data, 3)
								this.log(this.drawerData.DRUGINFO_NAME,2,0);
							}
						})
				} else {
					//新增
					var params = {
						"NAME": this.drawerData.DRUGINFO_NAME,
						"TID": this.drawerData.DRUGINFO_TID,
						"STOCK": this.drawerData.DRUGINFO_STOCK,
						"SUPPLIER": this.drawerData.DRUGINFO_SUPPLIER,
						"UID": Cookies.get("UUID"),
						"PHONE":this.drawerData.DRUGINFO_PHONE,
						"ISDELETE": "-2"
					}
					this.$http.insertData(this,
						"DRUGINFO",
						params,
						0, res => {
							if (res.data != '') {
								notices.notice(_this, '操作提示', '药品:' + this.drawerData.DRUGINFO_NAME + '新增成功。', 1)
								this.log(this.drawerData.DRUGINFO_NAME,1,1);
								//调用关闭，更新状态参数
								this.cancel();
							} else {
								notices.notice(_this, '操作提示', res.data == '' ? '意料之外的错误,请联系管理员' : res.data, 3)
								this.log(this.drawerData.DRUGINFO_NAME,1,0);
							}
						})
				}
			},
			//日志
			//名称，类型，状态
			log(name,type,status){
				//新增
				var params = {
					"LOGS": (type == 1 ? "新增" : type == 2 ? "更新（审核）" : "删除") + "药品:"+name+ (status == 1 ? "成功" : "失败"),
					"TYPE": type,
					"UID": Cookies.get("UUID"),
				}
				this.$http.insertData(this,
					"LOGINFO",
					params,
					0, res => {
						//..
					})
			},
			//打开编辑的抽屉
			edit(data, index) {
				this.isEdit = true
				this.getTypeList();
				this.drawerTap = '编辑药品'
				this.drawerRight = true
				this.drawerData = data.row
			},
			//打开新增的抽屉
			add() {
				this.isEdit = true
				this.getTypeList();
				this.drawerTap = '新增药品'
				this.drawerData = {}
				this.drawerRight = true
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
					"DRUGINFO_UID#EQ": this.formItem.uid == '0' ? null : Cookies.get("UUID"),
					'DRUGINFO_NAME#LIKE': this.formItem.serchName == '' ? null : '%' + this.formItem.serchName + '%',
					'W!DRUGINFO_ISDELETE#EQ': this.formItem.isDelete == '-1' ? null : this.formItem.isDelete,
				}

				this.$http.selectBase(this,
					"DRUGINFO,DTINFO1,SYSUSER1",
					"DRUGINFO",
					"DTINFO1_NAME,SYSUSER1_NAME",
					"DRUGINFO_CREATETIME",
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
				this.formItem.serchName = name;
				this.getTableData();
			},
			//设置Table的高度
			setTableHeight() {
				this.tableHeight = ($(window).height() - 233);
			},
			getTypeList(pid, data) {
				let params = {
					"DTINFO_ISDELETE#EQ": 0,
				}
				this.$http.selectBase(this,
					"DTINFO",
					"DTINFO",
					"",
					"DTINFO_CREATETIME",
					"2",
					1, 999,
					params, 0, res => {
						this.typeList = res.data.data;
					})
			},
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
