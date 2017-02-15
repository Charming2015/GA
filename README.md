# GA
js实现遗传算法，解决01背包问题
## js实现遗传算法解决背包问题。
- 首先你的去了解遗传算法是啥。这里有一个很好的入门文章：[遗传算法](http://baike.baidu.com/link?url=jjIIZW1FTplFglFx8vNw9Si-QHtAPNIx0HRtUpREv8qgI4r9Sm4S2S0HwsAtT611gQ9vAUpiOBZEAN4qim89j3DSuD86aGo56oEUhZC1YnlnHQ8K3b1TQpUC_XOT4Y8v "遗传算法")

- 还是自己百度吧~

### 遗传算法主要其实包括交配，变异，死亡，三个部分。
代码设计的话，主要包括：价值函数，交配，死亡，突变，完成性测试，这几个函数。


大家可以先看看这篇文章，敲一下里面的代码，对遗传算法有基本的了解。
[机器学习之Javascript篇：遗传算法介绍](http://www.cnblogs.com/wangweiqiang/p/5039525.html "机器学习之Javascript篇：遗传算法介绍")

我就是沿用上面这篇文章的基本框架，扩展成解决01背包问题的代码
创作过程中参考到了这篇文章：[遗传算法解背包问题（javascript实现）](https://segmentfault.com/a/1190000004989612)

我改进完代码之后，再新增了画图画表格展现结果的功能。

下面说一下我的代码

最简单产生基因类
- cost：表示分值。
- ptw：表示负重。
- code：表示基因
````javascript
var Gene = function(code){
	this.code = code;
	this.cost = 0;
	this.ptw = 0;
	this.MaxWeight = 0;
}
Gene.prototype.code = '';
````
价值函数，当携带重量大于最大重量时进行惩罚，把cost改为1，把cost返回出去是为了后面好比较得出最大的cost
````javascript
Gene.prototype.calcCost = function(MaxWeight){
	this.MaxWeight = MaxWeight;
	var WeightArr = this.WeightArr;
	var ValueArr = this.ValueArr;
	var totalWeight = 0;
	var totalValue = 0;
	for( let i = 0 ; i < this.code.length ; i++ ){
		// 1 表示携带这个东西 0表示不带这个东西
		if( this.code[i] == 1 ){
			totalWeight += WeightArr[i];
			totalValue += ValueArr[i];
		}
	}
	if( totalWeight > this.MaxWeight ){
		this.cost = 1;
	}else{
		this.cost = totalValue;
	}
	this.ptw = totalWeight;
	return this.cost;
}
````
交配函数，没啥好说的，`gene[0].mate(gene[1])`这种调用方法，会返回新的后代。
````javascript
Gene.prototype.mate = function(gene){
	var pivot = Math.round( Math.random() * this.Len );
	var child1 = this.code.substr(0 , pivot ) + gene.code.substr(pivot);
	var child2 = gene.code.substr(0 , pivot ) + this.code.substr(pivot);
	return [new Gene(child1) , new Gene(child2)];
}
````
突变函数，以变异概率作为参数。其中突变的长度和突变的位置尽量随机。
````javascript
// 突变函数
Gene.prototype.mutate = function(P_MUTATION){
	if( Math.random() > P_MUTATION ) return ;
	var newCode = this.code.split('');
	// 随机突变长度
	var changeLength = Math.floor( Math.random() * this.code.length );
	for(var i = 0 ; i < changeLength ; i ++){
		// 随机突变位置
		var index = Math.floor( Math.random() * this.code.length );
		newCode.splice( index , 1 , Math.round( Math.random() ) )
	}
	this.code = newCode.join("");
	
}
````
生成随机基因
````javascript
// 生成随机基因
Gene.prototype.random = function(){
	var temp = '';
	for(var i = 0 ; i < this.Len ; i ++ ){
		temp += Math.round( Math.random() );
	}
	this.code = temp;
}
````
接下来的比较复杂，请做好心理准备。
- times：表示计算出现最大值的次数
- allTimes：最大值在100（allTimes）代中都相同。这个是因为我有两种终止程序运行的方式（下面我会说）而设置的。
````javascript
var Population = function(pop){
	this.MaxWeight = pop.bag.maxW; //背包最大承重
	this.WeightArr = pop.bag.wArr;//物品重量
	this.ValueArr = pop.bag.vArr; //物品价值

	this.maxValue = 0;  //储存最大价值
	this.maxGene = [];  //储存最大价值组合

	this.times = 0;  //计算出现最大值的次数

	// 最大值在100（allTimes）代中都相同。
	this.allTimes = pop.similarGen == undefined ? 100 : pop.similarGen ;

	this.Size = pop.size == undefined ? 20 : pop.size ; //种群规模
	this.P_XOVER = 0.8;  //遗传概率
	this.P_MUTATION = pop.P_MUTATION == undefined ? 0.15 : pop.P_MUTATION ; ; // 变异概率
	this.generationNum = 0;  // 繁衍次数

	var self = this;

	// 这里补充写Gene的类，方便把WeightArr和ValueArr传入
	Gene.prototype.WeightArr = self.WeightArr
	Gene.prototype.ValueArr = self.ValueArr
	Gene.prototype.Len = self.WeightArr.length
	
	this.MaxGenerations = pop.maxGen == undefined ? '100' : pop.maxGen ; //总进化代数
	this.type = pop.type == undefined ? 'near100' : pop.type ; //终止函数的类型（是当有100代产生的最大值都相同时终止程序 还是 繁衍N代之后终止程序）

	this.members = []; // 种群所有对象

	// 生成Gene
	while( this.Size --){
		var gene = new Gene();
		gene.random();
		this.members.push(gene);
	}

}
````
按照分值进行排序
````javascript
Population.prototype.sort = function(){
	this.members.sort(function(a,b){
		return b.cost - a.cost;
	})
}
````
接下来又是复杂的，而且是最重要的部分，同志们打起精神
- mateNum：关于这个参数我想说一下的，mateNum表示交配的个体数。因为我没有故意去写一个选择算子函数（就是一个选择哪两个个体进行交配的函数），于是我用mateNum来设置得分排名最高的三个个体进行“杂交”产生后代。
- `self.members.splice( self.members.length - newChildNum , newChildNum )`：关于这里的话是这样的，我会直接裁掉排在后面几位的个体（死亡，或者叫优胜劣汰），然后把新的后代push在后面，然后计分，排序。。。。
- 完成性测试：我叫他终止程序。我这里写了两种终止程序的方式。
- 1> 其一是：near100（默认的方式），就是当繁衍的过程中有100代都出现了相同的结果时，终止程序。举个例子当繁衍到82代已经出现了最大值170，当第83~182代繁衍过程中都没有出现比170大的值的时候，就默认已经取得最大值了。可以终止程序了
- 2> 另一种方法就是指定繁殖多少代了。这个比较好理解。
- 我设置定时器 `setTimeout( function(){self.generation();} , 20);`的目的是防止爆炸性增长~
````javascript
Population.prototype.generation = function(){
	var self = this;
	// 计算分值
	for( let i = 0 ; i < self.members.length ; i ++ ){
		var curValue = self.members[i].calcCost(self.MaxWeight)
		if( curValue > self.maxValue){
			self.maxValue = curValue;
			self.maxGene = self.members[i].code;
			self.times = 0;
		}

	}
	// 根据分值排序
	self.sort();

	// 展示函数，结果即时体现
	self.display();
	
	// 交配个数
	var mateNum = 3;
	// 产生后代总数
	var newChildNum = mateNum*(mateNum-1)
	// 把种群末尾的淘汰掉，把新后代添加到种群
	self.members.splice( self.members.length - newChildNum , newChildNum )
	for(var i = 0 ; i < mateNum ; i ++){
		for(var j = i+1 ; j < mateNum ; j ++){
			var chi = self.members[i].mate( self.members[j] );
			self.members.push( chi[0] )
			self.members.push( chi[1] )
		}
	}	

	// 突变
	for( var i = 0 ; i < self.members.length ; i ++ ){
		self.members[i].mutate(  self.P_MUTATION );
		var curValue = self.members[i].calcCost(self.MaxWeight)
		if( curValue > self.maxValue){
			self.times = 0;
			self.maxValue = curValue;
			self.maxGene = self.members[i].code;
		}

		// 判断啥时候终止程序。
		if (self.type == 'maxGen' && self.generationNum >= self.MaxGenerations) {
			self.stop();
			return true;
		} else if (self.type == 'near100' && self.times == self.allTimes) { //连续100代不变就停止
			self.stop();
			return true;
		}

	}

	self.times++
	self.generationNum ++ ;
	
	setTimeout( function(){
		self.generation();
	} , 20);
}
````
终止时运行的程序。其实就是把最大值给交代好，然后画图画表格展示结果。
````javascript
Population.prototype.stop = function(){
	var self = this;
	if(maxmax < self.maxValue){
		maxmax = self.maxValue
	}
	// 辅助画图的。
	option.yAxis.max = 2*maxmax;
	option.series[0].data.push(self.maxValue);
	myChart.setOption(option);

	self.sort();
	self.display();
	perResult.push({maxV:self.maxValue,gene:self.maxGene })
	if(perResult.length == loopTimes){
		self.showTable(perResult)
 	}
}
````
最后想说一下fun函数的for循环的部分。这里循环操作可以理解为实验次数，因为循环（实验）一次很可能有偶然性的结果，不一定是最优的。
````javascript
for (var i = 0; i < loopTimes; i++) {
		option.xAxis.data.push(i+1);
		var population = new Population(pop);
		population.generation()
	}
````
后面的即时展现结果和画图画表格等函数就没必要说了，不是这里的重点。


下面说一下用法：(嗯嗯，代码里面也有，在基础的遗传算法的基础上增加了以及画图画表的功能而已个入口而已。)
````javascript
var pop = {
	// type:'maxGen',  //终止函数类型 ， 默认near100
	// maxGen:5,       //繁衍后代数（在type为'maxGen'时有用），默认100
	bag:{
		maxW:3000,     //背包限重
		wArr :  [700,600,1200,1000,800,200,500],  // 物品重量
		vArr : [10,40,30,50,35,40,30]   // 物品价值
	},
	// bag:{
	// 	maxW:30,
	// 	wArr : [3,5,7,10,15,20,25,6,8,2],
	// 	vArr : [10,6,5,8,4,6,12,3,2,1]
	// },
	loopTimes:10,   //循环次数（可以理解为实验次数）， 默认10
	// P_MUTATION:0.15,   //变异概率 ， 默认0.15
	similarGen:20,   //决定多少个后代的最大值相同时停止程序。（在type为near100时有用），默认100
	// size:30    //种群大小(建议20~100)，默认20
}
fun(pop);
````


我的代码里有很详细的注释，大家可以下载自行研究，不懂的部分欢迎评论。
根据这些代码我写了个好玩的程序。

### 假如你有3000块，去厦门的话你有40分（满分100）的可能遇到女神，但是要花费600块，去北京的话艳遇可能性提高到50分，但是要花费1000...你要怎样在有限的旅游预算之内，最大概率地偶遇你的女神呢？请看[遗传算法解决艳遇问题](http://chenhanming.com/project/GA/index.html "遗传算法解决艳遇问题")

感谢以下文章：
1. [机器学习之Javascript篇：遗传算法介绍](http://www.cnblogs.com/wangweiqiang/p/5039525.html "机器学习之Javascript篇：遗传算法介绍")

2. [遗传算法解背包问题（javascript实现）](https://segmentfault.com/a/1190000004989612)

















