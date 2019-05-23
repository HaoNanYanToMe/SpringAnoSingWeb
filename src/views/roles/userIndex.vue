<style lang="less">
	@import './dataSourceIndex.less';
</style>
<template>
	<div class="dataSource">
		<!-- 筛选 -->
		<Form inline :model="formItem" style="position:relative;width:100%;margin: 15px 0 0 0;">
			<FormItem>
				<span style="margin-left:15px;">检索内容 ：</span>
			</FormItem>
			<FormItem style="width:20%">
				<AutoComplete v-model="formItem.serchName" :clearable="clearable" icon="ios-search" placeholder="请输入要检索的用户姓名或企业登录名"
				  @on-change="searchName(formItem.serchName)"></AutoComplete>
			</FormItem>
			<FormItem>
				<span>分组 ：</span>
			</FormItem>
			<FormItem style="width:10%">
				<Select v-model="formItem.isDelete" @on-change="changeisDeleteForm" placeholder="请选择用户分组">
					<Option value="-1" key="-1">全部</Option>
					<Option value="9614475f90184fdab6549e1331055843" key="9614475f90184fdab6549e1331055843">普通用户</Option>
					<Option value="b5942a6c590546e59fcf4bd51b6a7c0f" key="b5942a6c590546e59fcf4bd51b6a7c0f">超级管理员</Option>
				</Select>
			</FormItem>
			<FormItem>
				<span style="margin-left:15px;">角色 ：</span>
			</FormItem>
			<FormItem style="width:10%">
				<Select v-model="formItem.isDelete" @on-change="changeisDeleteForm" placeholder="请选择用户角色">
					<Option value="-1" key="-1">全部</Option>
					<Option value="0" key="0">管理者</Option>
					<Option value="1" key="1">黑名单</Option>
					<Option value="2" key="2">普通用户</Option>
				</Select>
			</FormItem>
			<FormItem style="float:right;">
				<Button type="primary" @click="add">新增用户信息</Button>
			</FormItem>
		</Form>
		<Table :loading="loading" :highlight-row="highlight" @on-row-click="showTableData" stripe :columns="tableColumns"
		 :data="tableData" :height="tableHeight"></Table>
		<Page :total="totalItem" :page-size="pageSize" size="small" show-elevator show-total style="margin:15px 0 0 0;text-align: center;font-size: 14px;"
		 show-sizer :page-size-opts="sizeOpts" @on-change="changeNo" @on-page-size-change="changeSize"></Page>
		<!-- 抽屉-->
		<Drawer :title="drawerData.SYSUSER_PHONE" :closable="false" placement="left" width="650" v-model="drawerLeft">
			<CellGroup>
				<Divider>基本信息</Divider>
				<Cell title="姓名" :extra="drawerData.SYSUSER_NAME + ''" selected />
				<Cell title="部门" :extra="drawerData.SYSROLE_NAME == undefined ? '' : drawerData.SYSROLE_NAME + ''" />
				<Cell title="手机" :extra="drawerData.SYSUSER_MOBILE + ''" selected />
				<Cell title="状态" :extra="drawerData.SYSUSER_ISDELETE == 0 ? '管理者' : drawerData.SYSUSER_ISDELETE == 1 ?'黑名单' : '无登陆权限'" />
				<Cell title="最后登录IP" :extra="drawerData.SYSUSER_LASTIP == null ? '' :drawerData.SYSUSER_LASTIP + ''" selected />
				<Cell title="信息备注" :extra="drawerData.SYSUSER_NOTE + ''" selected />
			</CellGroup>
		</Drawer>
		<Drawer :title="drawerTap" placement="right" width="650" v-model="drawerRight" :mask-closable="false" @on-close="cancel"
		 :styles="styles">
			<Form :model="drawerData" ref="formValidate" :rules="ruleValidate" :label-width="80">
				<FormItem label="姓名" prop="SYSUSER_NAME">
					<Input v-model="drawerData.SYSUSER_NAME" placeholder="请输入当前用户的姓名"></Input>
				</FormItem>
				<FormItem label="登录名" prop="SYSUSER_EMAIL">
					<Input v-model="drawerData.SYSUSER_EMAIL" placeholder="请输入当前用户的企业登录名,如:zhangsan@xdf.cn"></Input>
				</FormItem>
				<FormItem label="密码" prop="SYSUSER_PASSWORD">
					<Input v-model="drawerData.SYSUSER_PASSWORD" type="password" placeholder="请输入当前用户的密码"></Input>
				</FormItem>
				<FormItem label="手机">
					<Input v-model="drawerData.SYSUSER_MOBILE" placeholder="请输入当前用户联系方式(手机),如:13800000000"></Input>
				</FormItem>
				<FormItem label="座机" prop="SYSUSER_PHONE">
					<Input v-model="drawerData.SYSUSER_PHONE" placeholder="请输入当前用户联系方式(座机),如:010-8788788"></Input>
				</FormItem>
				<FormItem label="权限" prop="SYSUSER_RID">
					<Select v-model="drawerData.SYSUSER_RID" :disabled="deptHidden" @on-change="getRole" style="width:220px"
					 placeholder="请选择权限">
						<Option value="b5942a6c590546e59fcf4bd51b6a7c0f" key="b5942a6c590546e59fcf4bd51b6a7c0f">超级管理员</Option>
						<Option value="9614475f90184fdab6549e1331055843" key="9614475f90184fdab6549e1331055843">普通用户</Option>
					</Select>
				</FormItem>
				<FormItem label="角色" prop="SYSUSER_ISDELETE">
					<RadioGroup v-model="drawerData.SYSUSER_ISDELETE">
						<Radio label="noEnble" @click="drawerData.SYSUSER_ISDELETE = 0">管理者</Radio>
						<Radio label="isEnble" @click="drawerData.SYSUSER_ISDELETE = 1">黑名单</Radio>
						<Radio label="passEnble" @click="drawerData.SYSUSER_ISDELETE = 2">普通用户</Radio>
					</RadioGroup>
				</FormItem>
				<FormItem label="信息备注">
					<Input v-model="drawerData.SYSUSER_NOTE" type="textarea" :autosize="{minRows: 2,maxRows: 6}" placeholder="请输入当前用户的备注信息,如:负责系统模块的运维"></Input>
				</FormItem>
			</Form>
			<div class="demo-drawer-footer">
				<Button style="margin-right: 8px" @click="cancel" ref='cancel'>取消操作</Button>
				<Button type="primary" @click="handleSubmit('formValidate')">确认操作</Button>
			</div>
		</Drawer>
		<Drawer :title="currentRole" placement="right" width="450" v-model="drawerRole" :mask-closable="false" @on-close="cancelRole"
		 :styles="styles">
			<Tree :data="TreeData" ref="tree" @on-select-change="changeRoleTap"></Tree>
			<div class="demo-drawer-footer">
				<Button style="margin-right: 8px" @click="cancelRole">取消操作</Button>
				<Button type="primary" @click="drawerRoleSub" :disabled="roleDisabled">确认操作</Button>
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
	</div>
</template>
<script>
	import notices from '../../notice.js'
	import moment from 'moment';
	import md5 from 'js-md5';

	import {
		formatDate
	} from '../formatDate.js'
	import Cookies from 'js-cookie';
	export default {
		data() {
			return {
				deptHidden: false,
				areaHidden: false,
				schoolHidden: false,
				//校验
				ruleValidate: {
					SYSUSER_NAME: [{
						required: true,
						message: ' ',
						trigger: 'blur'
					}],
					SYSUSER_PASSWORD: [{
						required: true,
						message: ' ',
						trigger: 'blur'
					}],
					SYSUSER_EMAIL: [{
						required: true,
						message: ' ',
						trigger: 'blur'
					}],
					SYSUSER_PHONE: [{
						required: true,
						message: ' ',
						trigger: 'blur'
					}],
					SYSUSER_RID: [{
						required: true,
						message: ' ',
						trigger: 'change'
					}],
					SYSUSER_ISDELETE: [{
						required: true,
						message: '请选择当前用户的角色',
						trigger: 'change'
					}],
				},
				countClick: 0,
				//模态框
				modalType: false,
				modelTap: '',
				modelId: '', //待确认删除的id
				modelVersion: '', //待确认删除的数据version
				//抽屉
				drawerLeft: false,
				drawerRight: false,
				drawerRole: false,
				drawerData: {},
				drawerTap: '编辑',
				currentRole: '',
				roleDisabled: true,
				//用于编辑及保存数据使用
				currentRoleId: '',
				isEdit: false,
				//部门list
				deptList: [],
				deptSerchList: [],
				styles: {
					height: 'calc(100% - 55px)',
					overflow: 'auto',
					paddingBottom: '53px',
					position: 'static'
				},
				TreeData: [],
				//筛选框
				clearable: true,
				formItem: {
					serchName: '',
					isDelete: "-1",
					dept: '',
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
				tableColumns: [{
						title: '姓名',
						key: 'SYSUSER_NAME'
					},
					{
						title: '登录名',
						key: 'SYSUSER_EMAIL'
					},
					{
						title: '权限',
						key: 'SYSROLE1_NAME'
					},
					{
						title: '角色',
						key: 'SYSUSER_ISDELETE',
						render: (h, params) => {
							return h('div', [
								h('span', {
									style: {
										color: params.row.SYSUSER_ISDELETE == 0 ? 'green' : params.row.SYSUSER_ISDELETE == 1 ? 'red' : 'grey'
									}
								}, params.row.SYSUSER_ISDELETE == 0 ? '管理者' : params.row.SYSUSER_ISDELETE == 1 ? '黑名单' : '普通用户')
							]);
						}
					}, {
						title: '最后访问时间',
						key: 'SYSUSER_DATETIME',
						render: (h, params) => {
							return h('div', [
								h('span', moment(params.row.SYSUSER_DATETIME).format("YYYY-MM-DD HH:mm:ss"))
							]);
						}
					}, {
						title: '操作',
						key: 'action',
						width: 200,
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
											this.currentRole = this.drawerData.SYSROLE1_NAME
											this.currentRoleId = this.drawerData.SYSUSER_RID
											this.drawerData.SYSUSER_ISDELETE = this.drawerData.SYSUSER_ISDELETE == 1 ? 'isEnble' :
												this.drawerData.SYSUSER_ISDELETE == 2 ? 'passEnble' : 'noEnble'
										}
									}
								}, '编辑'),
								h('Button', {
									props: {
										type: 'error',
										size: 'small'
									},
									on: {
										click: () => {
											this.isEdit = true
											this.modalType = true
											this.modelVersion = params.row.SYSUSER_VERSION + 1
											this.modelId = params.row.SYSUSER_ID
											this.modelTap = '您将删除用户：' + params.row.SYSUSER_NAME + ",数据一旦删除将无法恢复，是否确认无误?"
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
			//选择权限
			getRole(val) {

			},
			//选择性别
			changeisDeleteForm(val) {
				this.formItem.isDelete = val;
				this.getTableData();
			},
			//变更授权tap提示框数据
			changeRoleTap() {
				var checked = this.$refs.tree.getSelectedNodes();
				this.currentRole = checked.length > 0 ? checked[0].id != '0' ? checked[0].title : '未授权' : this.drawerData.SYSROLE1_NAME
				this.currentRoleId = checked.length > 0 ? checked[0].id : '0'
				this.roleDisabled = this.currentRole == '未授权' ? true : false
			},
			//删除方法
			remove() {
				var _this = this;
				this.$http.deleteForce(this,
					"SYSUSER",
					this.modelId,
					0, res => {
						if (res.data.search("成功") != -1) {
							//数据参数重置
							this.isEdit = false
							this.modalType = false
							this.getTableData()
							notices.notice(_this, '操作提示', '用户已删除成功。', 1)
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
			//关闭授权抽屉
			cancelRole() {
				this.drawerRole = false
			},
			//保存并同步已选的权限信息
			drawerRoleSub() {
				var checked = this.$refs.tree.getSelectedNodes();
				this.drawerData.SYSUSER_RID = checked.length > 0 ? checked[0].id : this.drawerData.SYSUSER_RID
				this.drawerData.SYSROLE1_NAME = checked.length > 0 ? checked[0].title : this.drawerData.SYSROLE1_NAME
				this.cancelRole()
			},
			handleSubmit(name) {
				this.$refs[name].validate((valid) => {
					if (valid) {
						this.drawerSubmit();
					}
				})
			},
			//信息保存
			drawerSubmit() {
				var _this = this;
				if (this.drawerData.SYSUSER_ID) {
					//修改
					var params = {
						"SYSUSER_NAME": this.drawerData.SYSUSER_NAME,
						"SYSUSER_EMAIL": this.drawerData.SYSUSER_EMAIL,
						"SYSUSER_MOBILE": this.drawerData.SYSUSER_MOBILE,
						"SYSUSER_PHONE": this.drawerData.SYSUSER_PHONE,
						"SYSUSER_PASSWORD": this.drawerData.SYSUSER_PASSWORD == " " ? null : md5(this.drawerData.SYSUSER_PASSWORD),
						"SYSUSER_ISDELETE": this.drawerData.SYSUSER_ISDELETE == 'isEnble' ? 1 : this.drawerData.SYSUSER_ISDELETE ==
							'passEnble' ? 2 : 0,
						"SYSUSER_RID": this.drawerData.SYSUSER_RID,
						"SYSUSER_NOTE": this.drawerData.SYSUSER_NOTE,
					}
					debugger;
					this.$http.updateData(this,
						"SYSUSER",
						params,
						this.drawerData.SYSUSER_VERSION + 1,
						this.drawerData.SYSUSER_ID,
						0, res => {
							debugger
							if (res.data.search("成功") != -1) {
								notices.notice(_this, '操作提示', '用户:' + this.drawerData.SYSUSER_NAME + '信息更新成功。', 1)
								//调用关闭，更新状态参数
								this.cancel();
							} else {
								notices.notice(_this, '操作提示', res.data == '' ? '意料之外的错误,请联系管理员' : res.data, 3)
							}
						})
				} else {
					var params = {
						"NAME": this.drawerData.SYSUSER_NAME,
						"EMAIL": this.drawerData.SYSUSER_EMAIL,
						"MOBILE": this.drawerData.SYSUSER_MOBILE,
						"PHONE": this.drawerData.SYSUSER_PHONE,
						"RID": this.drawerData.SYSUSER_RID,
						"NOTE": this.drawerData.SYSUSER_NOTE,
						"PASSWORD": md5(drawerData.SYSUSER_PASSWORD),
						"ISDELETE": this.drawerData.SYSUSER_ISDELETE == 'isEnble' ? 1 : this.drawerData.SYSUSER_ISDELETE ==
							'passEnble' ? 2 : 0,
					}
					this.$http.insertData(this,
						"SYSUSER",
						params,
						0, res => {
							if (res.data != '') {
								notices.notice(_this, '操作提示', '用户:' + this.drawerData.SYSUSER_NAME + '新增成功。', 1)
								//调用关闭，更新状态参数
								this.cancel();
							} else {
								notices.notice(_this, '操作提示', res.data == '' ? '意料之外的错误,请联系管理员' : res.data, 3)
							}
						})
				}
			},
			//打开编辑的抽屉
			edit(data, index) {
				this.isEdit = true
				this.drawerTap = '编辑系统用户信息'
				this.drawerRight = true
				this.drawerData = data.row
				this.drawerData.SYSUSER_PASSWORD = " "
			},
			//打开新增的抽屉
			add() {
				this.isEdit = true
				this.drawerTap = '新增系统用户信息'
				this.drawerData.SYSUSER_ISDELETE = 'noEnble'
				this.drawerData = {}
				this.drawerRight = true
			},
			//打开授权的抽屉
			addRole() {
				this.getTreeData()
				this.drawerRole = true
				this.currentRole = this.currentRole || '未授权'
			},
			//打开查询的抽屉
			showTableData(data, index) {
				if (!this.isEdit) {
					this.drawerLeft = true
					this.drawerData = data
					this.drawerData.SYSUSER_DATETIME = formatDate(new Date(data.SYSUSER_DATETIME),
						"yyyy-MM-dd hh:mm:ss")
				}
			},
			//获取Tree数据
			getTreeData() {
				var _this = this;
				let params = {}
				this.$http.exFunction(this,
					"sysRole",
					"getRoleTree",
					1, params, 0, res => {
						_this.TreeData = eval(res.data);
					})
			},
			//获取Table表单数据
			getTableData() {
				var _this = this;
				_this.loading = true;
				let params = {
					"WS!SYSUSER_NAME#LIKE": this.formItem.serchName == '' ? null : '%' + this.formItem.serchName + '%',
					"OE!SYSUSER_EMAIL#LIKE": this.formItem.serchName == '' ? null : '%' + this.formItem.serchName + '%',
					"W!SYSUSER_ISDELETE#EQ": this.formItem.isDelete == -1 ? null : this.formItem.isDelete,
					"W!SYSUSER_RID#EQ": this.formItem.dept == -1 ? null : this.formItem.dept,
				}
				this.$http.selectBase(this,
					"SYSUSER,SYSROLE1",
					"SYSUSER",
					"SYSROLE1_NAME",
					"SYSUSER_DATETIME",
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
