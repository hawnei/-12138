<template>
	<div id="InterviewWrap">
		<!-- 头部 -->
		<header class="header" style="font-size: 12px"> 
			<div>
        <!-- 项目 -->
				<span>类型</span>
				<el-select v-model="itype" placeholder="请选择">
					<el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
					</el-option>
				</el-select>
				<span>日期</span>
				<el-date-picker type="date" value-format="yyyy/MM/dd" placeholder="选择日期" v-model="icreateDate"></el-date-picker>
			</div>
			<div>
				<span>项目</span>
				<el-input placeholder="项目" prefix-icon="el-icon-search" v-model="ititle"></el-input>
				<span>项目</span>
				<el-input placeholder="项目" prefix-icon="el-icon-search" v-model="iCompanyname"></el-input>
				<el-button type="primary" icon="el-icon-search" @click="searchJL(search,3000);"   >搜索</el-button>
				<el-button type="primary" icon="el-icon-plus" @click="add">增加</el-button>
			</div> 
		</header>
		<el-container style="border: 1px solid #eee">
			<el-main>
				<!-- 标题 -->
				<div class="head">
					<el-row>
						<el-row>
							<el-col :span="1">序号</el-col>
							<el-col :span="2">类型</el-col>
							<el-col :span="7" align="left">问题标题</el-col>
							<el-col :span="2">创建日期</el-col>
							<el-col :span="3">项目</el-col>
							<el-col :span="3">项目</el-col>
							<el-col :span="2">创建用户</el-col>
							<el-col :span="3"> 操作 </el-col>
						</el-row>
					</el-row>
				</div>
				<!-- 列表 循环遍历 -->
				<el-collapse>
					<el-collapse-item v-for="(item,index) in arr" :key="index" :name="index+1" v-model="activeName" accordion>
						<template slot="title">
							<el-row>
								<el-col :span="1">
									<a :title="index+1">{{table_index(index)}}</a>&nbsp;
								</el-col>
								<el-col :span="2">
									<a :title="item.itype">{{item.itype}}</a>&nbsp;
								</el-col>
								<el-col :span="7">
									<a :title="item.title">{{item.title}}</a>&nbsp;
								</el-col>
								<el-col :span="2">
									<a :title="item.createDate">{{item.createDate | fmtDate}}</a>&nbsp;
								</el-col>
								<el-col :span="3">
									<a :title="item.company">{{item.company}}</a>&nbsp;
								</el-col>
								<el-col :span="3">
									<a :title="item.address">{{item.address}}</a>&nbsp;
								</el-col>
								<el-col :span="2">
									<a :title="item.createUser">{{item.createUser}}</a>&nbsp;
								</el-col>
								<el-col :span="3" align="right">
									<el-button type="primary" icon="el-icon-edit" circle @click.stop="edit(item)"></el-button>
									<el-button type="danger" icon="el-icon-delete" circle @click.stop="del(item)"></el-button>
								</el-col>
							</el-row>
						</template>
						<!-- 问题答案以及问题类型 -->
						<div class="answer">
							{{item.answers}}
						</div>
					</el-collapse-item>
				</el-collapse>
			</el-main>

		</el-container>
		<div v-if="paginationshow">
			<el-pagination   background layout="prev, pager, next" :current-page="currPage" :total="allSize" @current-change="changeCurrPage">
			</el-pagination>
		</div>
	</div>
</template>

<script>
	export default {
		name: "box",
		data() {
			return {
				paginationshow: true,
				currPage: 1,
				pagesize: 10,
				allSize: 1,
				activeName: "1",
				//搜索条件
				ititle: "",
				iCompanyname: "",
				icreateDate: "",
				//是否能够点击 
				startTime:Date.now(),
        timer:null,
        //与后台匹配
				arr: [{
					answers: "",
					company: "哈哈",
					address: "很长错错错错错错",
					createDate: "2019-12-20",
					createUser: "张三",
					id: 1,
					itype: "类型1",
					idesc: "",
					importantlevel: 0,
					sort: "",
					title: "你想问什么"
				}],
				options: [{ //下拉框
					value: '',
					label: ''
				}, { //下拉框
					value: '选项1',
					label: '选项1'
				}, {
					value: '选项2',
					label: '选项2'
				}],
				itype: ''
			};
		},
		filters: {
			fmtDate(val) {
				return val.substring(0, 10);
			}
		},
		mounted() {
			// this.search();
			this.searchJL(this.search,1000);
		},
		methods: {
			table_index(index) { 
				//显示行号
				return(this.currPage - 1) * this.pagesize + index + 1
			},
			btnsearch(){
				this.currPage = 1; 
				this.searchJL(this.search,1000);
      },
      //请求当前页数据
			getAllData(){
				return	this.$axios.get("api", {
						params: {
							page: this.currPage,
							pagesize: this.pagesize,
							title: this.ititle,
							companyname: this.iCompanyname,
							itype: this.itype,
							icreateDate:this.icreateDate
						}
					}) 
      },
      //总数
			getAllDataCount(){ 
				return		this.$axios.get("api", {
						params: {
							page: this.currPage,
							pagesize: this.pagesize,
							title: this.ititle,
							companyname: this.iCompanyname,
							itype: this.itype,
							icreateDate:this.icreateDate
						}
					}) 
			},
			searchJL(func,delay){ 
				var curTime = Date.now();             
				var remaining = delay - (curTime - this.startTime);                     
				clearTimeout(this.timer);              
				if (remaining <= 0) {                    
					func.apply(this, arguments);                    
					this.startTime = Date.now();              
				} else {                   
					this.timer = setTimeout(func, remaining);              
				}   
			},
			search() {  
					//需要同时调用多个后台接口，就会高并发的问题
					this.$axios.all([this.getAllData(),this.getAllDataCount()])
					.then(this.$axios.spread((resdata,rescount)=>{ 
						this.arr = resdata.data;
						this.allSize = rescount.data[0].count; 
					})) 
			},
			add() {
				this.$refs['EditDialogForm'].showDialog('新增', null, this.doCreate);
			},
			doCreate(data) {
				this.searchJL(this.search,1000);
			},
			edit(item) {
				const target = {
					title: item.title,
					idesc: item.idesc,
					itype: item.itype,
					answers: item.answers,
					createDate: item.createDate,
					createUser: item.createUser,
					sort: item.sort,
					importantlevel: item.importantlevel,
					company: item.company,
					address: item.address,
					id: item.id,
				};
				this.$refs['EditDialogForm'].showDialog('编辑', target, this.doEdit);
			},
			doEdit(data) {
				console.log('doedit');
				this.searchJL(this.search,1000);
			},
			//删除
			del(item) {
        this.$alert('是否要删除' + item.title + '?', '对象', {
					showCancelButton: true,
					confirmButtonText: '确定',
					callback: action => {
						if(action == "confirm") {
              //请求删除
							this.$axios.get("api", {
									params: {
										id: item.id
									}
								})
								.then(res => {
									this.$message({
										type: 'info',
										message: `删除成功`
									});
									this.searchJL(this.search,1000);
								})
								.catch(err => {
									console.log(err);
								});

						}

					}
				});

			},
			changeCurrPage(currPage) {
				this.currPage = currPage;
//				console.log("this.currPage",this.currPage) //点击第几页
				this.searchJL(this.search,1000);
			}
		}
	};
</script>

<style scoped lang="less">
	#InterviewWrap {
		.header {
			text-align: left;
			padding: 10px;
			>div{
				margin-bottom: 10px;
			}
			span{
				margin-right: 10px;
			}
		}
		.head {
			font-weight: bolder;
		}
		.el-collapse {
			border-bottom: none;
		}
		.el-col {
			text-align: left;
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
			a {
				text-decoration: none;
			}
		}
		.head {
			margin: 10px 0;
		}
		.el-input {
			width: 30%; 
		}
		.el-row {
			width: 100%;
		}
		.answer {
			text-align: left;
		}
	}
</style>