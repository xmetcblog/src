<template>
	<div>
		<div class="crumbs">
			<el-breadcrumb separator="/">
				<el-breadcrumb-item>
					<i class="el-icon-lx-cascades"></i> 基础表格
				</el-breadcrumb-item>
			</el-breadcrumb>
		</div>
		<div class="container">
			<div class="handle-box">
				<el-button type="primary" icon="el-icon-delete" class="handle-del mr10" @click="delAllSelection">批量删除</el-button>

				<!-- <el-select v-model="query.address" placeholder="地址" class="handle-select mr10">
                    <el-option key="1" label="广东省" value="广东省"></el-option>
                    <el-option key="2" label="湖南省" value="湖南省"></el-option>
                </el-select>
                <el-input v-model="query.name" placeholder="用户名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button> -->
			</div>
			<el-table :data="page.list" border class="table" ref="multipleTable" header-cell-class-name="table-header"
			 @selection-change="handleSelectionChange">

				<el-table-column type="selection" width="55" align="center"></el-table-column>
				<el-table-column prop="uid" label="ID" width="55" align="center"></el-table-column>
				<el-table-column label="用户名">
					{{nickName}}
				</el-table-column>
				<!--                <el-table-column label="账户余额">
                    <template slot-scope="scope">￥{{scope.row.money}}</template>
                </el-table-column>
                <el-table-column label="头像(查看大图)" align="center">
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.thumb"
                            :preview-src-list="[scope.row.thumb]"
                        ></el-image>
                    </template>
                </el-table-column> -->
				<el-table-column prop="title" label="标题"></el-table-column>
				<el-table-column label="状态" align="center">
					<template slot-scope="scope">
						<el-tag :type="scope.row.state==='成功'?'success':(scope.row.state==='失败'?'danger':'')">{{scope.row.state}}</el-tag>
					</template>
				</el-table-column>

				<el-table-column prop="editTime" label="修改时间"></el-table-column>
				<!-- 操作先置后 -->
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
			<el-form ref="form" :data="form" label-width="70px">
				<el-form-item label="文章标题">
					<el-input v-model="form.title"></el-input>
				</el-form-item>
				<el-form-item label="文章总结">
					<el-input v-model="form.summary"></el-input>
					<el-button icon="el-icon-edit" @click="gotoSummary">修改文章内容</el-button>
				</el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="editVisible = false">取 消</el-button>
				<el-button type="primary" @click="saveEdit">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
	import {
		fetchData
	} from '../../api/index';
	var axios = require('axios');
	export default {
		name: 'basetable',
		data() {
			return {
				role: localStorage.getItem('role'),
				query: {
					address: '',
					name: '',
					pageIndex: 1,
					pageSize: 10
				},
				uid: localStorage.getItem('id'),
				nickName: localStorage.getItem('nickName'),
				role: localStorage.getItem('role'),
				page: {},
				tableData: [],
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
		/* created() {
		    this.getData();
		} */
		mounted: function() {
			this.getData();
		},
		methods: {
			// 获取 easy-mock 的模拟数据
			/* getData() {
			    fetchData(this.query).then(res => {
			        console.log(res);
			        this.tableData = res.list;
			        this.pageTotal = res.pageTotal || 50;
			    });
			}, */
			//获取后台文章数据
			getData: function(pageNum, pageSize) {
				var role = this.role;
				axios.get("http://localhost:8763/article/PageAllArticleOrCon" + "?pageNum=" + this.pageNum, {
					//if (role) {
						params: {
							uid: role==0?null:this.uid
					//	}
					}
				}).then((response) => {
					this.page = response.data;
					this.pageTotal = this.page.total
					console.log(this.page)
				})
			},
			//修改后台文章数据
			editArticle: function() {
				axios.get("http://localhost:8763/article/updateArticle", {
					params: {
						id: this.id,
						title: this.page.list[this.idx].title
					}
				}).then((response) => {
					console.log(response)
				})
			},
			//删除文章
			deleteArticle: function() {
				axios.get("http://localhost:8763/article/deleteArticle", {
					params: {
						id: this.id
					}

				}).then((response) => {
					console.log(response)
				})
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
						this.deleteArticle();
						this.$message.success('删除成功');
						this.tableData.splice(index, 1);
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
			handleEdit(index, form) {
				this.idx = index;
				this.id = form.id;
				this.form = JSON.parse(JSON.stringify(form));
				console.log(form)
				this.editVisible = true;
			},
			// 保存编辑
			saveEdit() {
				this.editVisible = false;
				this.$message.success(`修改第 ${this.idx + 1} 篇成功`);
				this.$set(this.page.list, this.idx, this.form);
				this.editArticle();
			},
			// 分页导航
			handlePageChange(val) {
				this.pageNum = val,
					console.log(this.pageNum),
					this.getData();
			},
			gotoSummary() {
				localStorage.setItem('aid',this.id);
				this.$router.push('/articlemark');
			}
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
