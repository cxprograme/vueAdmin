<template>
	<el-container>
		<el-aside width='200px'> 
			<el-row>
				<lazy-tree :parent-node="pNode" @treedata='getTreeData' @pmenuid='getPmenuId' :page-size="pagesize" ></lazy-tree>
			</el-row>
		</el-aside>
		<el-container>
			<el-header>
				<span style="float:right;"> 
                	<el-button type="text" @click="add" style="color:white">添加</el-button>  
                	<el-button type="text" @click="deletenames" style="color:white">批量删除</el-button>        
            	</span> 
			</el-row>
			</el-header>
			<el-main>
				<el-input placeholder="请输入内容" v-model="criteria" style="padding-bottom:10px;">
		              <el-select v-model="select" slot="prepend" placeholder="请选择">
		                 <el-option label="id" value="1"></el-option>
		                 <el-option label="name" value="2"></el-option>
		              </el-select>
		              <el-button slot="append" icon="search" v-on:click="search"></el-button>
	          	</el-input>
	          	<el-table
		            ref="testTable"       
		            :data="tableData.slice((currentPage-1)*pagesize,currentPage*pagesize)"
		            style="width:100%"
		            border
		            :default-sort = "{prop: 'id', order: 'ascending'}"
		            @selection-change="handleSelectionChange"   
		            @row-click="handleclick"
		            :row-class-name = "tableRowClassName"
		            >
		            <el-table-column
				      prop="date"
				      label="日期"
				      sortable
				      width="180">
				    </el-table-column>
				    <el-table-column
				      prop="menu_id"
				      label="页面号"
				      sortable
				      width="180">
				    </el-table-column>
				    <el-table-column
				      prop="address"
				      label="地址"
				      :formatter="formatter">
				  </el-table-column>
		            <el-table-column label="操作">
		              <template scope="scope">
		                <el-button
		                  size="small"
		                  type="primary"
		                  @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
		                <el-button
		                  size="small"
		                  type="danger"
		                  @click="handleDelete(scope.$index, scope.row)">删除</el-button>
		              </template>
		            </el-table-column>
		          </el-table>
				 <el-pagination
                  @size-change="handleSizeChange"
                  @current-change="handleCurrentChange"
                  :current-page="currentPage"
                  :page-sizes="[5,10,20, 30, 40]"
                  :page-size="pagesize"
                  layout="total, sizes, prev, pager, next, jumper"
                  :total="totalCount">
              </el-pagination>
			</el-main>
            <el-dialog title="菜单信息" :visible.sync="dialogFormVisible">
              <el-form :model="form">
                <el-form-item label="页面号" :label-width="formLabelWidth">
                  <el-input v-model="form.menu_id" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="JS" :label-width="formLabelWidth">
                  <el-input v-model="form.js" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="Procedure" :label-width="formLabelWidth">
                  <el-input v-model="form.procedure" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="Lcr.xml" :label-width="formLabelWidth">
                  <el-input v-model="form.lcrxml" auto-complete="off"></el-input>
                </el-form-item>
              </el-form>
              <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
              </div>
            </el-dialog>
		</el-container>
    </el-container>
    <!-- <div >pNode {{typeof(pNode)}}:{{pNode}} </div> -->
	<!-- </div> -->
    
</template>
<script type="text/javascript">
	import LazyTree from '@/components/LazyTree'
    import $ from 'jquery'
	export default {
          data(){       
          	return {
          		pNode:[],


          		 //表格当前页数据
                tableData: [],
                //树节点的id
                pMenuId:'',

                //多选数组
                multipleSelection: [],

                //请求的URL
                url:'http://localhost:8081/menuwork/menus/admin',

                //搜索条件
                criteria: '',

                //下拉菜单选项
                select: '',

                //默认每页数据量
                pagesize: 5,

                //默认高亮行数据id
                highlightId: -1,

                //当前页码
                currentPage: 1,

                //查询的页码
                start: 1,

                //默认数据总数
                totalCount: 1000,
                dialogFormVisible: false,
                form: {
                  menu_id: '',
                  js:'',
                  procedure:'',
                  lcrxml:''
                },
          	}
               
            },
            components:{
            	LazyTree
            },
            mounted(){
            	var _this = this;
            	_this.axios.get('http://localhost:8081/menuwork/menus/admin/tree',{params:{
            		'menu_type':'0,1',
        			'user_id':'default_menu',
        			'parent_menu_id':'userRoot'
            	}}).then(function(res){
            		// this.pNode = JSON.stringify(res.data);
            		_this.pNode = res.data;
            		// _this.pNode[0]['sub_menu'][0]['sub_menu'] = [{'menu_name':'测试','menu_id':'1212'}];
            		console.log('this pNode:' + Object.prototype.toString.call(res.data));

            		
        		   // console.log('this pNode:' + JSON.stringify(_this.pNode))         			

            	})
            },
            methods: {
            	getTreeData:function(msg){
            		// console.log('父组件：'+msg)
                    console.log("分页信息:"+ JSON.stringify(msg));  
                    console.log("页面数:"+ msg.pageSize);
                    this.tableData = msg.list;
                    this.totalCount = msg.total;
                    this.pagesize = msg.pageSize;
                    this.currentPage = msg.pageNum;
                    console.log("pagesize:"+this.pagesize);
            	},
                getPmenuId:function(msg){
                    this.pMenuId = msg;
                },
                //从服务器读取数据
                loadData: function(criteria, pageNum, pageSize){
                    var _this = this;    
                    $.ajax({
                    url:'http://localhost:8081/menuwork/menus/admin',
                    type:'GET', //GET
                    async:false,    //或false,是否异步
                    data:{
                        'parameter':criteria, 
                        'pageNum':pageNum, 
                        'pageSize':pageSize,
                        'menu_type':'0,1',
                        'user_id':'default_menu',
                        'parent_menu_id':_this.pMenuId
                    },
                    timeout:5000,    //超时时间
                    dataType:'json',    //返回的数据格式：
                    beforeSend:function(xhr){
                    },
                    success:function(data,textStatus,jqXHR){
                        _this.tableData = data.list;
                        _this.totalCount = data.total;
                        _this.pagesize = data.pageSize;
                        _this.currentPage = data.pageNum;
                    },
                    error:function(xhr,textStatus){
                    },
                    complete:function(){
                    }
                })
                   /* this.axios.get('http://localhost:8081/menuwork/menus/admin',{params:{
                         'parameter':criteria, 
                        'pageNum':pageNum, 
                        'pageSize':pageSize,
                        'menu_type':'0,1',
                        'user_id':'default_menu',
                        'parent_menu_id':this.pMenuId
                    }}).then(function(res){
                        console.log()
                        this.tableData = msg.list;
                        this.totalCount = msg.total;
                        this.pagesize = msg.pageSize;
                        this.currentPage = msg.pageNum;
                    })          */       
                    /*this.$http.get(this.url,
                        {
                            'parameter':criteria, 
                            'pageNum':pageNum, 
                            'pageSize':pageSize,
                            'menu_type':'0,1',
                            'user_id':'default_menu',
                            'parent_menu_id':this.pMenuId
                        }).then(function(res){
                            // this.tableData = res.data.pagestudentdata;
                            // this.totalCount = res.data.number;
                            this.tableData = msg.list;
                            this.totalCount = msg.total;
                            this.pagesize = msg.pageSize;
                            this.currentPage = msg.pageNum;
                    },function(){
                        console.log('failed');
                    });      */           
                },

                //多选响应
                handleSelectionChange: function(val) {
                    this.multipleSelection = val;
                },

                //点击行响应
                handleclick: function(row, event, column){
                    this.highlightId = row.id;
                },

                //编辑
                handleEdit: function(index, row) {
                    // this.$prompt('请输入新名称', '提示', {
                    //       confirmButtonText: '确定',
                    //       cancelButtonText: '取消',
                    //     }).then(({ value }) => {
                    //         if(value==''||value==null)
                    //             return;
                    //         this.$http.post('newstu/update',{"id":row.id,"name":value},{emulateJSON: true}).then(function(res){
                    //             this.loadData(this.criteria, this.currentPage, this.pagesize);                              
                    //         },function(){
                    //             console.log('failed');
                    //         });
                    //     }).catch(() => {

                    // });
                    this.dialogFormVisible = true;
                    this.form.menu_id = row.menu_id;
                    this.form.procedure = 'p_lcr_cfg_financManaing';
                    this.form.lcrxml = 'rest/lcr';
                    this.form.js = 'FinanceInfoInput';
                    // console.log(JSON.stringify(row));
                },


                //单行删除
                handleDelete: function(index, row) {
                    // var array = [];
                    // array.push(row.id);
                    // this.$http.post('newstu/delete',{"array":array},{emulateJSON: true}).then(function(res){
                    //     this.loadData(this.criteria, this.currentPage, this.pagesize);
                    // },function(){
                    //     console.log('failed');
                    // });
                },

                //搜索
                search: function(){
                    this.loadData(this.criteria, this.currentPage, this.pagesize);
                },

                //添加
                add: function(){
                    //     this.$prompt('请输入名称', '提示', {
                    //       confirmButtonText: '确定',
                    //       cancelButtonText: '取消',
                    //     }).then(({ value }) => {
                    //         if(value==''||value==null)
                    //             return;
                    //         this.$http.post('newstu/add',{"name":value},{emulateJSON: true}).then(function(res){
                    //             this.loadData(this.criteria, this.currentPage, this.pagesize);
                    //         },function(){
                    //             console.log('failed');
                    //         });
                    //     }).catch(() => {

                    // });

                },

                //多项删除
                deletenames: function(){
                    // if(this.multipleSelection.length==0)
                    //     return;
                    // var array = [];
                    // this.multipleSelection.forEach((item) => {
                    //     array.push(item.id);
                    // })
                    // this.$http.post('newstu/delete',{"array":array},{emulateJSON: true}).then(function(res){
                    //     this.loadData(this.criteria, this.currentPage, this.pagesize);
                    // },function(){
                    //     console.log('failed');
                    // });
                },

                //改变当前点击的行的class，高亮当前行
                tableRowClassName: function(row, index){
                   if(row.id == this.highlightId)
                   {
                      return 'info-row';
                   }
                },

                //每页显示数据量变更
                handleSizeChange: function(val) {
                    this.pagesize = val;
                    this.loadData(this.criteria, this.currentPage, this.pagesize);
                },

                //页码变更
                handleCurrentChange: function(val) {
                    this.currentPage = val;
                    this.loadData(this.criteria, this.currentPage, this.pagesize);
                },
            }
	}
</script>
<style rel="stylesheet/scss" lang="scss">
	.el-header{
		background-color: #eee;
		color:#000;
		line-height: 60px;
	}

</style>