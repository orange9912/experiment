<template>
  <div>
	  <el-form :inline="true" :model="form" class="demo-form-inline">
	    <el-form-item label="算法">
	      <el-select v-model="form.type" placeholder="请选择">
	          <el-option
	            v-for="item in options"
	            :key="item.name"
	            :label="item.name"
	            :value="item.name">
	          </el-option>
	        </el-select>
	    </el-form-item>
		<el-form-item v-show="form.type == '电梯SCAN' || form.type == 'C-SCAN'" label="初始方向">
			<el-select v-model="form.isBigger" placeholder="请选择">
			    <el-option label="往增大方向" value="1"></el-option>
				<el-option label="往减小方向" value="0"></el-option>
			</el-select>
		</el-form-item>
	    <el-form-item label="磁头初始位置">
	      <el-input v-model="form.initPosition" ></el-input>
	    </el-form-item>
	    <el-form-item>
	      	  <el-button type="primary" @click="updateList">更新原始随机数据</el-button>
			  <el-button type="primary" @click="updateA">更新算法</el-button>
			  <!-- <el-button type="primary" @click="test"></el-button> -->
	    </el-form-item>
	  </el-form>
	<div id="myChart" :style="{width: '1000px', height: '500px'}"></div>
	<label>平均寻道长度：{{averageLength}}</label>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data:function(){
	  return{
		  options:[
			  {name:'FCFS'},
			  {name:'SSTF'},
			  {name:'电梯SCAN'},
			  {name:'C-SCAN'}
		  ],
		  numberList:[],//原始生成数据
		  ServerList:[],//真正服务顺序
		  FlagList:[],
		  times:[],//次数从1~400
		  form:{
			  type:'',//算法
			  initNumbers:1500,//磁盘磁道数
			  initPosition:0,//磁头初始位置
			  isBigger:'',
		  },
		  averageLength:0,
	  }
  },
  mounted(){
	  for(let i = 0;i<=400;i++)
	  {
		  this.times.push(i)
	  }
      this.updateList()
   },
   methods: {
       drawLine(){
           // 基于准备好的dom，初始化echarts实例
           let myChart = this.$echarts.init(document.getElementById('myChart'))
           // 绘制图表
           myChart.setOption({
               title: { text: '磁盘调度算法的模拟实现与对比' },
               tooltip: {},
               xAxis: {
                   data:this.times 	
               },
			   dataZoom:[{
				   type:'slider',
				   start:1,
				   end:400,
			   },
			   {   // 这个dataZoom组件，也控制x轴。
			       type: 'inside', // 这个 dataZoom 组件是 inside 型 dataZoom 组件
			       start: 1,  
			       end: 400 
			   }
			   ],
               yAxis: {name:'服务磁道号'},
               series: [{
                   name: '此次的磁道号',
                   type: 'line',
                   data: this.ServerList
               }]
           });
       },
	   updateList:function(){
		   //随机生成磁道号序列，共400个
		   //50%位于0~499,25%分布于50~999，25%分布于1000~1499
		   //先清空一次数组
		   this.numberList = []
		   this.FlagList = []
		   let flag1 = 0,flag2 = 0, flag3 = 0//各个位置生成数
		   for(;flag1+flag2+flag3<400;)//生成400个
		   {
			   let area = this.randomNum(0,3)//三个区域随机选择，限制生成个数
			   let num = 0
			   if((area === 0 || area === 1) && flag1 <200)//
			   {
				   num = this.randomNum(0,499)
				   this.numberList.push(num)
				   this.FlagList.push(false)
				   flag1++;
			   }
			   if(area === 2 && flag2 <100)
			   {
				   num = this.randomNum(500,999)
				   this.numberList.push(num)
				   this.FlagList.push(false)
				   flag2++;
			   }
			   if(area === 3 && flag3 <100)
			   {
				   num = this.randomNum(1000,1499)
				   this.numberList.push(num)
				   this.FlagList.push(false)
				   flag3++;
			   }
		   }
		   //更新数据同时也要重新利用算法计算
		   this.updateA()
	   },
	   //生成从minNum到maxNum的随机数
	   randomNum:function(minNum,maxNum){ 
	       switch(arguments.length){ 
	           case 1: 
	               return parseInt(Math.random()*minNum+1,10); 
	           break; 
	           case 2: 
	               return parseInt(Math.random()*(maxNum-minNum+1)+minNum,10); 
	           break; 
	               default: 
	                   return 0; 
	               break; 
	       } 
	   },
	   
	   updateA:function(){
		   this.ServerList = []//先清空原始数据栏
		   //然后flag全部置false
		   for(let i = 0;i<this.FlagList.length;i++)
		   {
			   this.FlagList[i] = false
		   }
		   this.averageLength = 0
		   this.ServerList.push(this.form.initPosition)//从磁头初始位置开始
		   if(this.form.type == "FCFS")
		   {
			   this.FCFS()//执行该算法
		   }
		   if(this.form.type == "SSTF")
		   {
			   this.SSTF()
		   }
		   if(this.form.type == "电梯SCAN")
		   {
			   this.SCAN()  
			} 
		   if(this.form.type == "C-SCAN")
		   {
			   this.C_SCAN()
		   }
		   this.drawLine()
	   },
	   FCFS:function(){
		   //先来先服务 即按照数组下标进行顺序服务
		   let position = this.form.initPosition // 磁头当前位置
		   for(let i = 0;i<this.numberList.length; i++)
		   {
			   this.averageLength += Math.abs(position - this.numberList[i])
			   position = this.numberList[i]//将当前磁头位置修正到刚服务完的地方
			   this.ServerList.push(this.numberList[i])//先服务的压入数据
			   this.FlagList[i] = true//并标记为已服务，按照原始顺序
		   }
		   this.averageLength /= this.numberList.length//计算平均寻道长度
	   },
	   
	   SSTF:function(){
		   //最短寻道 循环查找，找到距离最短的再动
		   //需要标记哪些已经服务
		   let position = this.form.initPosition // 磁头当前位置
		   let distance = 1500,index = 0//当前最短距离，及最短距离的下标记录
		   for(let i = 0;i<this.numberList.length;i++)//共长度一样个数据要
		   {
			   distance = 1500
			   index = 0
			   //先循环查找最短的、且未被标记的
			   for(let j = 0;j<this.numberList.length;j++)
			   {
				   let nowDistance = Math.abs(position - this.numberList[j])
				   if(this.FlagList[j] == false && nowDistance<=distance)//未被服务且距离更小的才计算
				   {
					   distance = nowDistance//更新最短距离
					   index = j//记录下标
				   }
			   }
			   //此时已找到最短的
			   this.ServerList.push(this.numberList[index])//最短的 服务了 压入
			   this.FlagList[index] = true //标记已经服务
			   this.averageLength += distance
			   position = this.numberList[index]//修正
		   }
		   this.averageLength /= this.numberList.length
	   },
	   SCAN:function(){
		   //只往增大或减小方向
		   let position = this.form.initPosition
		   let flag = this.form.isBigger //1为增大
		   // alert(typeof flag)
		   let distance = 1500,index = -1//当前最短距离，及最短距离的下标记录
		   for(let i = 0;i<this.numberList.length;i++)//共长度一样个数据
		   {
		   	   distance = 1500
		   	   index = -1
		   	   //先循环查找最短的、且未被标记的
		   	   for(let j = 0;j<this.numberList.length;j++)
		   	   {
		   		   let nowDistance = this.numberList[j] - position//当前磁头位置-这个位置，大于0即在更高，小于0在低
				   if(flag === '1' && nowDistance >=0)
				   {//增大方向、且找到不小于当前位置的才服务 
						nowDistance = Math.abs(nowDistance)//记录绝对值距离
					   if(this.FlagList[j] == false && nowDistance<=distance)//未被服务且距离更小的才计算
					   {
					   	  distance = nowDistance//更新最短距离
					  	  index = j//记录下标
					   }
				   }
				   if(flag === '0' && nowDistance <=0)
				   {//减小方向、且找到不大于当前位置的才服务 
				   	 nowDistance = Math.abs(nowDistance)//记录绝对值距离
				   	if(this.FlagList[j] == false && nowDistance<=distance)//未被服务且距离更小的才计算
				   	{
				   		distance = nowDistance//更新最短距离
						index = j//记录下标
				    }
				   }
				   // nowDistance = position - this.numberList[j]
				   
		   	   }
		   	   //此时已找到最短的
			   if(index == -1)//即没找到
			   {
				   if(flag == '1')
				   {
					   flag = '0'
				   }
				   else 
				   {
					   flag = '1'
				   }
				   i--
				   continue
			   }
		   	   this.ServerList.push(this.numberList[index])//最短的 服务了 压入
		   	   this.FlagList[index] = true //标记已经服务
		   	   this.averageLength += distance
		   	   position = this.numberList[index]//修正
		   }
		   this.averageLength /= this.numberList.length
	   },
	   C_SCAN:function(){
		   //只往增大或减小方向
		   let position = this.form.initPosition
		   let flag = this.form.isBigger //1为增大
		   // alert(typeof flag)
		   let distance = 1500,index = -1//当前最短距离，及最短距离的下标记录
		   for(let i = 0;i<this.numberList.length;i++)//共长度一样个数据
		   {
		   	   distance = 1500
		   	   index = -1
		   	   //先循环查找最短的、且未被标记的
		   	   for(let j = 0;j<this.numberList.length;j++)
		   	   {
		   		   let nowDistance = this.numberList[j] - position//当前磁头位置-这个位置，大于0即在更高，小于0在低
				   if(flag === '1' && nowDistance >=0)
				   {//增大方向、且找到不小于当前位置的才服务 
						nowDistance = Math.abs(nowDistance)//记录绝对值距离
					   if(this.FlagList[j] == false && nowDistance<=distance)//未被服务且距离更小的才计算
					   {
					   	  distance = nowDistance//更新最短距离
					  	  index = j//记录下标
					   }
				   }
				   if(flag === '0' && nowDistance <=0)
				   {//减小方向、且找到不大于当前位置的才服务 
				   	 nowDistance = Math.abs(nowDistance)//记录绝对值距离
				   	if(this.FlagList[j] == false && nowDistance<=distance)//未被服务且距离更小的才计算
				   	{
				   		distance = nowDistance//更新最短距离
						index = j//记录下标
				    }
				   }
				   
		   	   }
		   	   //若到末尾，则回到最前的磁道处再往后
			   let lest = 1500,lest_index = -1
			   if(flag == '1')
			   {
				   lest = 1500
			   }
			   else{
				   lest = -1
			   }
			   if(index == -1)//即没找到
			   {//则去到另一面尽头最大/最小的那个继续当前方向
				   for(let j = 0;j<this.numberList.length;j++)
				   {
				   		if(flag === '1')
				   		{//若是增大方向则去找最小的
				   			if(this.FlagList[j] == false && this.numberList[j]<lest)
				   			{
				   				lest = this.numberList[j]
				   				lest_index = j//记录下标
				   			}
				   		}
				   		if(flag === '0')
				   		{//若是减小方向则去找最大的
				   			if(this.FlagList[j] == false && this.numberList[j]>lest)//未被服务且距离更小的才计算
				   			{
				   				lest = this.numberList[j]
				   				lest_index = j//记录下标
				   			}
				   		} 
				   }
					index = lest_index//找到了最大/最小的让其
					distance = Math.abs(this.numberList[index] - position)
			   }
		   	   this.ServerList.push(this.numberList[index])//最短的 服务了 压入
		   	   this.FlagList[index] = true //标记已经服务
		   	   this.averageLength += distance
		   	   position = this.numberList[index]//修正
		   }
		   this.averageLength /= this.numberList.length
	   }
},
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#myChart{
	margin-left: 200px;
}
</style>
