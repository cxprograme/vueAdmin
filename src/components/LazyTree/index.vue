<template>
	<!-- <el-tree
	  :props="props1"
	  :load="loadNode1"
	  lazy
	  @node-click='clickNode'
	  >
	</el-tree> -->
	<el-tree
	  :data="parentNode"
	  :props="props1"
	  accordion
	  ref='tree'
	  lazy
	  :load="loadNode"
	  @node-click="handleNodeClick">
	</el-tree>
</template>
<script>
	import $ from 'jquery';
	export default {
		name:'lazyTree',
		props:{
			parentNode:{
				type:Array,
				default:function(){
					return []
				}
			},
			pageNum:{
				type:Number,
				default:1
			},
			pageSize:{
				type:Number,
				default:4
			}
		},
	    data() {
	      return {
	        props1: {
	          label: 'menu_name',
	          children: 'sub_menu',
	          isLeaf: 'leaf'
	        },
	         defaultProps: {
	          children: 'sub_menu',
	          label: 'menu_name'
	        }
	      };
	    },
	    methods: {
	      loadNode:function(node,resolve){
	      	if (node.level === 0) {
         		return resolve(this.parentNode);
        	}
        	if (node.level > 3) return resolve([]);
        	if (node.level >0) {
	        	 this.getSub(node.data.menu_id,node.data.menu_name,resolve);

        	}
	      },

	      getSub:function(menu_id,menu_name,resolve){
	      	// alert(JSON.stringify(data));
	      	/*if(data.sub_menu){
	      		// alert(data.sub_menu);
	      		// resolve(data.sub_menu);
	      	}else{
	      		alert('没有值了');
	      	}*/
	      	var _this = this;
        	_this.axios.get('http://localhost:8081/menuwork/menus/admin/tree',{params:{
            		'menu_type':'0,1',
        			'user_id':'default_menu',
        			'parent_menu_id':menu_id
        	}}).then(function(res){
        		// console.log('res DATA:'+res.data);
        		resolve(res.data);
        	}) 
	      },
	      loadParent:function(){

	      },
	      handleNodeClick:function(data,node){
	      	var _this = this;
	      	if(node.level == 4){
	      		/*$.ajax({
				    url:'http://localhost:8081/menuwork/menus/admin/tree',
				    type:'GET', //GET
				    async:false,    //或false,是否异步
				    data:{
				        'menu_type':'0,1',
				        'user_id':'default_menu',
        				'parent_menu_id':node.data.menu_id
				    },
				    timeout:5000,    //超时时间
				    dataType:'json',    //返回的数据格式：
				    beforeSend:function(xhr){
				    },
				    success:function(data,textStatus,jqXHR){
				    	// console.log('Data Table'+data);
				    	_this.$emit('treedata',data);
				    },
				    error:function(xhr,textStatus){
				    },
				    complete:function(){
				    }
				})*/
				$.ajax({
				    url:'http://localhost:8081/menuwork/menus/admin',
				    type:'GET', //GET
				    async:false,    //或false,是否异步
				    data:{
				        'menu_type':'0,1',
				        'user_id':'default_menu',
        				'parent_menu_id':node.data.menu_id,
        				'pageNum':_this.pageNum,
        				'pageSize':_this.pageSize
				    },
				    timeout:5000,    //超时时间
				    dataType:'json',    //返回的数据格式：
				    beforeSend:function(xhr){
				    },
				    success:function(data,textStatus,jqXHR){
				    	// console.log('Data Table'+data);
				    	_this.$emit('treedata',data);
				    	_this.$emit('pmenuid',node.data.menu_id);
				    },
				    error:function(xhr,textStatus){
				    },
				    complete:function(){
				    }
				})
	      	}
			// console.log(node);
		    	// if(node.level == 4){
	      	// 	alert(node);
	      	// }
	      	/*console.log(this.$refs.tree.getCurrentNode());
	      	var node =this.$refs.tree.getCurrentNode();

	      	console.log("menu_id:" + node.menu_id)
	      	if(node.menu_id=='1000024'){
	      		console.log('node-menu-id:' + node);
	      		var pNode = node.parent_menu_id;
	      		var ret2 = this.parentNode.findIndex((v) => {
				    return v.menu_id == pNode;
				});
				var node = this.parentNode[ret2];
				if(node.hasOwnProperty('sub_menu')){
					if(this.parentNode[ret2]['sub_menu'][0]['sub_menu'])
						this.parentNode[ret2]['sub_menu'][0]['sub_menu'].push({'menu_name':'测试','menu_id':'1212'})

					else{
						// this.parentNode[ret2]['sub_menu'][0]['sub_menu'] = [{'menu_name':'测试','menu_id':'1212'}];
						 this.$set(this.parentNode[ret2]['sub_menu'][0], 'sub_menu', [{'menu_name':'测试','menu_id':'1212'}]);
					}

				}else{
					this.parentNode[ret2]['sub_menu'] = [{'menu_name':'测试','menu_id':'1212'}];
				}
	      	}*/
	      }
	    },

  	};
</script>
<style rel="stylesheet/scss" lang="scss">
	
</style>