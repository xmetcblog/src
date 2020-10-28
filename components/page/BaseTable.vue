<template>
	<div>
		<div class="crumbs">
			<el-breadcrumb separator="/">
				<el-breadcrumb-item>
					<i class="el-icon-lx-cascades"></i> 用户管理
				</el-breadcrumb-item>
			</el-breadcrumb>
		</div>
		<div class="container">
			<div class="handle-box">
				<el-button type="primary" icon="el-icon-delete" class="handle-del mr10" @click="delAllSelection">批量删除</el-button>
				<el-select v-model="query.address" placeholder="是否可用" class="handle-select mr10" @change="selectUsers">
					<el-option key="1" label="正常用户" value="1"></el-option>
					<el-option key="2" label="封禁用户" value="0"></el-option>
				</el-select>
				<el-input v-model="query.name" placeholder="用户名" class="handle-input mr10"></el-input>
				<el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
				<span style="float: right;">
					<el-button type="primary" @click="dialogVisible  = true">添加用户</el-button>
				</span>
			</div>
			<el-table :data="userList" border class="table" ref="multipleTable" header-cell-class-name="table-header"
			 @selection-change="handleSelectionChange">
				<el-table-column type="selection" width="55" align="center"></el-table-column>
				<el-table-column prop="id" label="编号" width="55" align="center"></el-table-column>
				<el-table-column prop="userName" label="用户名"></el-table-column>
				<el-table-column prop="nickName" label="昵称">
				</el-table-column>
				<el-table-column label="头像(查看大图)" align="center">
					<template slot-scope="scope">
						<el-image class="table-td-thumb" :src="scope.row.userFace" :preview-src-list="[scope.row.userFace]"></el-image>
					</template>
				</el-table-column>
				<el-table-column prop="email" label="email"></el-table-column>
				<el-table-column prop="enabled" label="状态" align="center">
					<!-- <template slot-scope="scope">
						<el-tag :type="scope.row.state==='成功'?'success':(scope.row.state==='失败'?'danger':'')">{{scope.row.state}}</el-tag>
					</template> -->
					<template slot-scope="scope">
						<el-switch v-model="scope.row.enabled" active-color="#13ce66" inactive-color="#ff4949" :active-value="1"
						 :inactive-value="0" @change="changeStatus($event,scope.row,scope.$index)" />
					</template>
				</el-table-column>

				<el-table-column prop="regTime" label="注册时间"></el-table-column>
				<el-table-column label="操作" width="180" align="center">
					<template slot-scope="scope">
						<el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
						<el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
			<div class="pagination">
				<el-pagination background layout="total, prev, pager, next" :current-page="pageNum" :page-size="5" :total="pageTotal"
				 @current-change="handlePageChange"></el-pagination>
			</div>
		</div>

		<!-- 编辑弹出框 -->
		<el-dialog title="编辑" :visible.sync="editVisible" width="30%">
			<el-form ref="userList" :model="userList" label-width="70px">
				<el-input type="hidden" v-model="userList.id"></el-input>
				<el-form-item label="用户名">
					<el-input v-model="userList.userName"></el-input>
				</el-form-item>
				<el-form-item label="昵称">
					<el-input v-model="userList.nickName"></el-input>
				</el-form-item>
				<el-form-item label="邮箱">
					<el-input v-model="userList.email"></el-input>
				</el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="editVisible = false">取 消</el-button>
				<el-button type="primary" @click="saveEdit">提交</el-button>
			</span>
		</el-dialog>
		<!-- 添加弹出框 -->
		<el-dialog title="添加用户" :visible.sync="dialogVisible" width="30%" :before-close="handleClose">
			<el-form ref="userList" :model="userList" label-width="70px">
				<el-form-item label="用户名">
					<el-input v-model="userList.userName"></el-input>
				</el-form-item>
				<el-form-item label="昵称">
					<el-input v-model="userList.nickName"></el-input>
				</el-form-item>
				<el-form-item label="密码" prop="pass">
				    <el-input type="password" v-model="userList.password" auto-complete="off"></el-input>
				  </el-form-item>
				  <el-form-item label="确认密码" prop="checkPass">
				    <el-input type="password" v-model="userList.checkpassword" auto-complete="off"></el-input>
				  </el-form-item>
				<el-form-item label="邮箱">
					<el-input v-model="userList.email"></el-input>
				</el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="dialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="insertUser()">提交</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
	import {
		fetchData
	} from '../../api/index';
	var axios = require('axios')
	export default {


		name: 'basetable',
		data() {
			return {
				dialogVisible: false,
				query: {
					address: '',
					name: '',
					pageIndex: 1,
					pageSize: 10
				},
				userList: [{
					id: '',
					userName: '',
					nickName: '',
					password: '',
					enabled: '',
					email: '',
					userFace: '',
					regTime: '',
					role: ''
				}],
				multipleSelection: [],
				delList: [],
				editVisible: false,
				pageTotal: 0,
				pageNum: 1,
				form: {},
				idx: -1,
				id: -1
			};
		},
		created() {
			this.getData();
		},
		methods: {

			//获取用户列表
			getUserList: function(pageNum, pageSize) {
				axios.get("http://localhost:8762/login/pageUserByCondition", {
					params: {
						// uid: this.uid,
						pageNum: this.pageNum
					}
				}).then(response => {
					var userList = response.data.list;
					var page = response.data;
					console.log(userList);
					if (userList) {
						this.userList = userList,
							this.pageTotal = page.total,
							this.pageNum = page.pageNum
					}
				}, response => {
					console.log(response);
					alert("服务器未打开！")
				});
			},
			//改变表格中的el-switch开关状态
			changeStatus(e, row, index) { //e表示el-switch的状态（true，false）
				//请求接口
				console.log(e);
				axios.get('http://localhost:8762/login/UpUserState', {
					params: {
						id: row.id,
						enabled: e,
					}
				}).then(res => {
					console.log('切换状态成功');
					//TODO ：刷新列表数据

				}).catch(err => {
					console.log('切换状态失败');
					let newData = row;
					newData.status = newData.status === 1 ? 0 : 1; //恢复 原状态
					this.userList[index] = newData;
				})
			},
			//查找
			selectUsers(value) {
				//请求接口
				console.log("enabled状态" + value)
				axios.get('http://localhost:8762/login/pageUserByCondition', {
					params: {
						enabled: value,
					}
				}).then(res => {
					var userList = res.data.list;
					var page = res.data;
					console.log(userList);
					if (userList) {
						this.userList = userList,
							this.pageTotal = page.total,
							this.pageNum = page.pageNum
					}

				}).catch(err => {
					console.log('切换状态失败');
					let newData = row;
					newData.status = newData.status === 1 ? 0 : 1; //恢复 原状态
					this.userList[index] = newData;
				})
			},

			//删除用户
			deleteUser: function() {
				axios.get("http://localhost:8762/login/delUser", {
					params: {
						id: this.id
					}
				}).then((response) => {
					console.log(response)
				})
			},

			// 获取 easy-mock 的模拟数据
			getData() {
				fetchData(this.query).then(res => {
					console.log(res);
					this.tableData = res.list;
					this.pageTotal = res.pageTotal || 50;
				});
			},
			// 触发搜索按钮
			handleSearch() {
				this.$set(this.query, 'pageIndex', 1);
				this.getData();
			},
			// 删除操作
			handleDelete(index, form) {
				// 二次确认删除
				this.$confirm('确定要删除吗？', '提示', {
						type: 'warning'
					})
					.then(() => {
						this.id = form.id;
						this.deleteUser();
						this.$message.success('删除成功');
						this.userList.splice(index, 1);
					})
					.catch(() => {
						this.$message.error('放弃删除');
					});
			},
			// 多选操作
			handleSelectionChange(val) {
				this.multipleSelection = val;
			},
			delAllSelection() {
				const length = this.multipleSelection.length;
				let str = '';
				this.delList = this.delList.concat(this.multipleSelection);
				for (let i = 0; i < length; i++) {
					str += this.multipleSelection[i].name + ' ';
				}
				this.$message.error(`删除了${str}`);
				this.multipleSelection = [];
			},
			// 编辑操作
			handleEdit(index, row) {
				this.idx = index;
				this.userList = row;
				this.editVisible = true;
			},
			handleClose(done) {
				this.$confirm('确认关闭？')
					.then(_ => {
						done();
					})
					.catch(_ => {});
			},

			// 保存编辑
			saveEdit() {
				this.editVisible = false;
				this.$message.success(`修改第 ${this.idx + 1} 行成功`);
				this.$set(this.userList, this.idx, this.userList);
				axios.get("http://localhost:8762/login/updateUser", {
					params: {
						id: this.userList.id,
						userName: this.userList.userName,
						nickName: this.userList.nickName,
						email: this.userList.email,
					}
				}).then((response) => {
					this.getUserList()
				})
			},
			//添加用户
			insertUser(){
				axios.get("http://localhost:8762/login/addUser", {
					params: {
						id: this.userList.id,
						userName: this.userList.userName,
						nickName: this.userList.nickName,
						email:this.userList.password,
						email: this.userList.email,
					}
				}).then((response) => {
					this.getUserList()
				})
			},
			// 分页导航
			handlePageChange(val) {
				this.pageNum = val,
					this.getUserList();
			}
		},
		created() {
			this.getUserList()
		}
	};
</script>

<style scoped>
	.handle-box {
		margin-bottom: 20px;
	}

	.handle-select {
		width: 120px;
	}

	.handle-input {
		width: 300px;
		display: inline-block;
	}

	.table {
		width: 100%;
		font-size: 14px;
	}

	.red {
		color: #ff0000;
	}

	.mr10 {
		margin-right: 10px;
	}

	.table-td-thumb {
		display: block;
		margin: auto;
		width: 40px;
		height: 40px;
	}
</style>
