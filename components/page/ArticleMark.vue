<template>
	<div>
		<div class="crumbs">
			<el-breadcrumb separator="/">
				<el-breadcrumb-item><i class="el-icon-lx-calendar"></i> 表单</el-breadcrumb-item>
				<el-breadcrumb-item>markdown编辑器</el-breadcrumb-item>
			</el-breadcrumb>
		</div>
		<div class="container">
			<div class="plugins-tips">
				mavonEditor：基于Vue的markdown编辑器。
				访问地址：<a href="https://github.com/hinesboy/mavonEditor" target="_blank">mavonEditor</a>
			</div>
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
	var axios = require('axios');
	export default {
		name: 'markdown',
		data: function() {
			return {
				content: '',
				html: '',
				configs: {},
				aid: localStorage.getItem('aid'),
				article: {},
				id: -1,
				mdContent: '',
				htmlContent: ''
			}
		},
		components: {
			mavonEditor
		},
		created: function() {
			this.getData();
		},
		methods: {
			//根据文章id查询文章
			getData: function(pageNum, pageSize) {
				axios.get("http://localhost:8763/article/findArticleById", {
					params: {
						id: this.aid
					}
				}).then((response) => {
					this.article = response.data;
					this.content = this.article.mdContent
					console.log(this.article)
				})
			},
			updateArticle: function() {
				console.log(this.content);
				console.log(this.html);				
				axios.get("http://localhost:8763/article/updateArticleById", {
					params: {
						id: this.aid,
						mdContent: this.content,
						htmlContent: this.html
					}
				}).then((response) => {
					console.log(response)
				})
			},
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
			submit() {
				console.log(this.content);
				console.log(this.html);
				this.updateArticle();
				this.$message.success('提交成功！');
			}
		}
	}
</script>
<style scoped>
	.editor-btn {
		margin-top: 20px;
	}
</style>
