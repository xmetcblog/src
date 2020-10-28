<template>
	<div>
		<div class="crumbs">
			<el-breadcrumb separator="/">
				<el-breadcrumb-item><i class="el-icon-lx-calendar"></i> 表单</el-breadcrumb-item>
				<el-breadcrumb-item>markdown编辑器</el-breadcrumb-item>
				<br />
				<div align="center">
				<el-select v-model="categoryList.id" placeholder="请选择文章栏目" style="width: 150px;">
					<el-option v-for="item in categoryList" :key="item.id" :label="item.cateName" :value="item.id">
					</el-option>
				</el-select>
				<el-input v-model="articleData.title" pla
				ceholder="请输入标题..." style="width: 400px;margin-left: 10px"></el-input>
				</div>
			</el-breadcrumb>
		</div>
		<div class="container">
			<!--  <div class="plugins-tips">
                mavonEditor：基于Vue的markdown编辑器。
                访问地址：<a href="https://github.com/hinesboy/mavonEditor" target="_blank">mavonEditor</a>
            </div> -->
			<mavon-editor v-model="content" ref="md" @imgAdd="$imgAdd" @change="change" style="min-height: 600px" />
			<el-button class="editor-btn" type="primary" @click="submit">提交</el-button>
		</div>
	</div>
</template>

<script>
	import {
		mavonEditor
	} from 'mavon-editor'
	import 'mavon-editor/dist/css/index.css'
	var axios = require('axios')
	export default {
		name: 'markdown',
		data: function() {
			return {
				articleData: {
					id: '',
					title:'',
					mdContent: '',
					htmlContent: '',
					summary: '',
					cid: '',
					uid: '',
					publishDate: '',
					editTime: '',
					state: '1',
					pageView: ''
				},
				categoryList: [{
					id: '',
					cateName: '',
					date: ''
				}],
				content: '',
				html: '',
				configs: {}
			}
		},
		components: {
			mavonEditor
		},
		methods: {
			// 将图片上传到服务器，返回地址替换到md中
			$imgAdd(pos, $file) {
				var formdata = new FormData();
				formdata.append('file', $file);
				// 这里没有服务器供大家尝试，可将下面上传接口替换为你自己的服务器接口
				this.$axios({
					url: '/common/upload',
					method: 'post',
					data: formdata,
					headers: {
						'Content-Type': 'multipart/form-data'
					},
				}).then((url) => {
					this.$refs.md.$img2Url(pos, url);
				})
			},
			change(value, render) {
				// render 为 markdown 解析后的结果
				this.html = render;
			},
			submit: function() {
				axios.get('http://localhost:8763/article/addArticle', {
						params: {
							title:this.articleData.title,
							mdContent: this.content,
							htmlContent: this.html,
							cid:this.categoryList.id,
							state: this.articleData.state
						}
					})
					.then(response => {
						console.log(this.content);
						console.log(this.html);
						this.$message.success('提交成功！');
					}, response => {
						console.log(response);
						alert("服务器未打开！")
					});
			},
			getCategory: function() {
				axios.get('http://localhost:8763/category/queryCategoryAll')
					.then(response => {
						var category = response.data;
						console.log(category);
						if (category) {
							this.categoryList = category
						}
					}, response => {
						console.log(response);
						alert("服务器未打开！")
					});
			}
		},
		created() {
			this.getCategory(),
			this.getPageAllArticle()
		}
	}
</script>
<style scoped>
	.editor-btn {
		margin-top: 20px;
	}
</style>
