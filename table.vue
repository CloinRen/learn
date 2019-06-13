ifhire
<template>
	<div>
		<el-form :inline="true" :model="searchForm" class="search-form" >
			<el-form-item label="商业别名">
				<el-input v-model="searchForm.name" class="form-width7" placeholder="请输入商业别名" ></el-input>
			</el-form-item>
			<el-form-item>
				<el-button type="primary" @click="onSearch" icon="el-icon-search">查询</el-button>
				<el-button type="primary" @click="Clean"><span class="fa fa-eraser left-icon"></span>清除</el-button> 
				<el-button type="primary" class="fr" :loading="setLoading" @click="setFormSubmit"><span class="fa fa-save left-icon"></span>保存</el-button>
			</el-form-item>
		</el-form>
		<div class="table-box border-radius-all setList">
			<!-- @row-click = "showRow"  -->
			<el-table :data="tableData" v-loading="listLoading" stripe border ref="multipleTable"  @selection-change="selectionChangeHandle" max-height="400">
				<el-table-column  type="selection" width="55" align="center" fixed></el-table-column>
				 <!--
				 <el-table-column  width="55" align="center" fixed>
					<template slot-scope="scope">
					  <el-radio  v-model="radio"  :label="scope.row.confirmid" @change.native="getTemplateRow(scope.$index,scope.row)">&nbsp;</el-radio> 
					</template>
				</el-table-column>
				-->
				<!-- <el-table-column prop="businesscode" label="商业编码"  align="cenetr"  width="260"></el-table-column> -->
				<el-table-column prop="name" label="商业别名"  align="cenetr"  width="320"  fixed></el-table-column>
				<el-table-column prop="provincename" label="省份" width="130"  align="center" ></el-table-column>
				<el-table-column prop="city" label="城市" width="130" align="center"></el-table-column>
				<el-table-column  label="商业名称"  align="cenetr"  >
					<template slot-scope="scope">
						<el-radio-group v-model="dataListSelections[scope.$index]" >
							<el-radio :label="item.aiId"    @change.native="getTemplateRow(scope.$index,scope.row)" v-for="(item,index) in scope.row.radioList" :key="index"  >
							  <span class="liks">{{item.subname }}</span>	
							</el-radio>
						</el-radio-group>
						
					</template>
				</el-table-column>
			</el-table>
			
			<div class="table-paginate-box clearfloat border-radius-bottom">
				<el-pagination class="fr"
							@size-change="handleSizeChange"
							@current-change="handleCurrentChange"
							:current-page="currentPage"
							:page-sizes="[10, 20, 30, 50,100]"
							:page-size="pageSize"
							layout="total, sizes, prev, pager, next, jumper"
							:total="total">
				</el-pagination>
			</div>
		</div>
	</div>

</template>

<script>
	
	export default {
		data(){
			return {
				listLoading: false,
				searchForm:{
					name:'',
					
				},
				currentPage:1, //页码定义
				total:0,
				pageSize:10,
				radio: '',
 				tableData:[],
				listData:[],
				setLoading:false,
				selected:[],
				dataListSelections:[] ,
// 				selectedID:"",
// 				selectedIDs:[],
				
			}
		},
		methods:{	
			selectionChangeHandle(val) {

							let ids = [];
							let idss = [];
							val.map((item) => {
								let confirmId = item.confirmid;
								ids.push(item.confirmid);
								// console.log(item.radioList);
								item.radioList.map((a) =>{
										// a.aiId;
										// console.log(this.dataListSelections);
										this.dataListSelections.map((b) =>{
											if(a.aiId==b){
												idss.push(confirmId + "-" +a.aiId);
											}
										});
										
								});
								
							})
						
							this.selected = idss;
							// console.log(this.selected);
			
		
	},
		
// 		showRow(row){
// 			//赋值给radio
// 			// this.radio = this.tableData.indexOf(row);
// 			let index=this.tableData.indexOf(row);
// 			this.radio=row.confirmid;
// 			this.selectedID=this.dataListSelections[index];
// 			// this.selected=row;
// 
// 		},
			getTemplateRow(index,row){
			let confirmId = row.confirmid;
			
			if(this.selected.filter((a) =>{
				return a.split('-')[0] == confirmId;
			}).length ==0){
				return;
			}
			this.selected = this.selected.filter((a) =>{
				return a.split('-')[0] != confirmId;
			})
		
		
			row.radioList.map((a) =>{
					this.dataListSelections.map((b) =>{
						if(a.aiId==b){
								this.selected.push(confirmId +"-" + a.aiId);
						}
					});
					
			});
			// console.log(this.selected);
			// this.selectionChangeHandle(row);
// 				this.radio=row.confirmid;
// 
//  				if(this.tableData[index].confirmid==this.radio && this.dataListSelections[index]!=undefined){
//  					
//  					this.selectedID=this.dataListSelections[index];
//  					
//   				}else{
//   					return	this.$alert("请选择当前行所对应的别名");
//  				}
// 				console.log("id----"+this.radio+"selectedID---"+this.selectedID)
			
			},
			// 获取所有别名列表
			getSetList() {
				this.listLoading= true
				var url = "businessAi/queryBusinessConfirmAlias"
				this.$api.post(url, {
					'limit': this.pageSize, 
					'offset': this.currentPage,
					'paging':1,
					'name': this.searchForm.name
				}, (res) => {     
						if (res.status == 200) {
							this.listLoading = false;
							this.tableData = res.data.dto.rows;
							this.total = res.data.dto.total;
							if(res.data.firstRadios!=[]||res.data.firstRadios!=null ||res.data.firstRadios!=undefined){
								this.arrList(res.data.firstRadios);
							}else{
								this.dataListSelections=[];
							}
							
							
							
						} else {
							this.listLoading = false;
							return false;
						}
			
				})
			},
			setFormSubmit() {
			
				
				if(this.selected.length==0){
					return	this.$alert("请选择要保存的行数据");
				}
				
				this.setLoading = true;
				let param = {
					'datas': this.selected
				}
				
				this.$api.post('businessAi/addBusinessAliasByBatch',param, (res) => {
					if (res.status == 200) {
						this.setLoading = false;
						this.$message({
							message: res.msg,
							type: 'success'
						});
						this.$refs.multipleTable.clearSelection();
						this.getSetList();
					} else {
						this.setLoading = false;
						return false;
					}
				});
			},
			
			//查询
			onSearch(){
				if(this.searchForm.name==''){
					this.currentPage=1;
					this.getSetList();
				}else{
					this.getSetList();
				}
				
				
			},
			
			Clean(){ //清空功能
				this.searchForm.name='';
				this.currentPage=1;
				this.getSetList();
			},
			
			//改变分页大小
			handleSizeChange(val) {
				this.pageSize = val;
				this.currentPage=1;
				this.getSetList();
			},
			//跳转页数
			handleCurrentChange(val) {
				this.currentPage = val;
				this.getSetList();
			},
			arrList(arr){
				let ids = [];
				arr.map((item) => {
					ids.push(item.aiId)
				})
				this.dataListSelections = ids;
			}
			
			
						
			
		},
		mounted() {
			this.getSetList();
			
//  			let ids = []
//   			this.listData.map((item) => {
//  				ids.push(item.aiId)
//  			})
//   			this.dataListSelections = ids;
 			
			
		}
	}
</script>

<style>
.setList .el-radio-group label{
	margin-right: 1em;
	width: 100% !important;
}
.setList .el-radio+.el-radio {
    margin-left: 0em;
}
.liks{
	white-space: pre-line;
	
}
</style>
