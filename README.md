# first-week
This code is just to practice multi-chart linkage and learn to call each other functions.
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>github第一周的任务</title>
		<link rel="stylesheet" href="css/github_1.css" />
	    <script type="text/javascript" src="js/d3.js" charset="UTF-8"></script>
	</head>
	<body>
		<div class="container">
			<h1>课程考核成绩分布</h1>
			<div id="left_frame">
				
			</div>
			<div id="right_frame">
				
			</div>
			<div id="last_frame">
				
			</div>
		</div>

<script>	
function dashboard(fData){
	
	  function color(c){ 
	        	return {failed:'#A61D00',Pass:"#FF6140", medium:"#FF2C00",excellent:"#FF8B73"}[c]; }
	        var barColor = '#61D7A4';
		     //计算所有状态的分段总频率
			var margin={left: 50, top: 40, right: 10, bottom: 10}
			fData.forEach(function(d){d.total=d.grade.failed+d.grade.Pass+d.grade.medium+d.grade.excellent;});
			//第一个div，直方图
		function histoGram(fD){
			var hG={};
			var height=350;
	        var width=500;
			var svg_1=d3.select('#left_frame').append('svg')
			    .attr('width', width)
				.attr('height', height);
			
			var main=svg_1.append('g')
			    .attr('transform', "translate(" + (margin.left-15)+ ',' + (margin.bottom+10) + ')');     
			//定于x。y轴的比例尺
//			console.log(fData)
			var xScale = d3.scale.ordinal()
				.domain(fD.map(function(d){return d[0];}))
				.rangeRoundBands([0, width - margin.left-margin.right],0.1);
				
			var yScale = d3.scale.linear()
			    .range([height-margin.top-margin.bottom, 0])
                .domain([0, d3.max(fD,function(d) {return d[1];})]);
            //创建x，y轴    
            var xAxis = d3.svg.axis()
				.scale(xScale)
				.orient('bottom');
			var yAxis = d3.svg.axis()
				.scale(yScale)
				.orient('left');
			//添加x轴，一般放在最后为好		
		    main.append('g')
				.attr('class', 'axis')
				.attr('transform', 'translate('+0+',' +(height-margin.top)+ ')')
				.call(xAxis);
//这里不需要将y轴画出来，因为y轴的比例尺是会变动的，想要y轴也是可以实现的
//		    main.append('g')
//				.attr('class', 'axis')
//				.attr('transform', 'translate('+0+',' +margin.bottom+ ')')
//				.call(yAxis);
			//创建矩形	
			var minpadding = 4;
			
	  var bars = main.selectAll(".bar").data(fD).enter()
                   .append("g").attr("class", "bar");
			 bars.append("rect")
				.attr("x", function(d) { return xScale(d[0]); })
                .attr("y", function(d) { return yScale(d[1]); })
                .attr("width", xScale.rangeBand()- minpadding)
                .attr("height", function(d) { return height - margin.top - yScale(d[1]); })
				.style('fill', barColor)
				.on("mouseover",mouseover)
                .on("mouseout",mouseout);
                
            function mouseover(d){  // 在鼠标悬停上调用实用功能，筛选选定的状态。
					 var st = fData.filter(function(s){ return s.course == d[0];})[0],
					 nD = d3.keys(st.grade).map(function(s){
                	     return {type:s, grade:st.grade[s]};});
                	 
			        pC.update(nD);
                    leg.update(nD);
			        }
            function mouseout(d){// 在mouseout上调用的实用函数。 重置饼图和图例。    
		            pC.update(tF);
                    leg.update(tF);
        }
            
            //创建矩形的文字标签
          bars.append("text").text(function(d){ return d3.format(",")(d[1])})
            .attr("x", function(d) { return xScale(d[0])+xScale.rangeBand()/2; })
            .attr("y", function(d) { return yScale(d[1])-5; })
            .attr("text-anchor", "middle");
		
        hG.update = function(nD, color){
                
            yScale.domain([0, d3.max(nD, function(d) { return d[1]; })]);
            var bars = main.selectAll(".bar").data(nD);
            bars.select("rect").transition().duration(500)
                .attr("y", function(d) {return yScale(d[1]); })
                .attr("height", function(d) { return height -margin.top- yScale(d[1]); })
                .style("fill", color);
             
            // transition the gradeuency labels location and change value.
            bars.select("text").transition().duration(500)
                .text(function(d){ return d3.format(",")(d[1])})
                .attr("y", function(d) {return yScale(d[1])-5; });            
        }        
        return hG;
    }
		
        function pieChart(pD){
        	var pC ={};
			//第二个div 饼图
			var height_1=350;
	        var width_1=300;
	      
			var svg_2=d3.select('#right_frame').append('svg')
			.attr('width', width_1)
			.attr('height', height_1).append("g")
			.attr('transform', "translate(" + margin.left*3+ ',' + margin.top*4+ ')');

		    var pie = d3.layout.pie()
			.sort(null)
			.value(function(d) {
				return d.grade;
			});
			var innerradius = 0;
			var outerradius = width_1 / 3;
			//创建弧生成器
			var arc = d3.svg.arc()
			    .innerRadius(innerradius)
				.outerRadius(outerradius);
			svg_2.selectAll("path").data(pie(pD)).enter().append("path").attr("d", arc)
            .each(function(d) { this._current = d; })
            .style("fill", function(d) { return color(d.data.type); })
            .on("mouseover",mouseover).on("mouseout",mouseout);

        // create function to update pie-chart. This will be used by histogram.
        pC.update = function(nD){
            svg_2.selectAll("path").data(pie(nD)).transition().duration(500)
                .attrTween("d", arcTween);
        }        
             function mouseover(d){
             hG.update(fData.map(function(v){ 
                return [v.course,v.grade[d.data.type]];
             }),color(d.data.type));
//            console.log(color(d.data.type));   
        }
          
            function mouseout(d){
            hG.update(fData.map(function(v){
                return [v.course,v.total];}), barColor);
        }
            
          function arcTween(a) {
			        var i = d3.interpolate(this._current, a);
			        this._current = i(0);
			        return function(t) { return arc(i(t));    };
        }    
               return pC; 
     }   
			
            //第三个：legend
   function legend(lD){
   	    var leg = {};
		var svg_3=d3.select('#last_frame')
					.append("table")
					.attr('class','legend');
	//创建行
var tr =svg_3.append("tbody").selectAll("tr").data(lD).enter().append("tr");
		
        // 第一列：矩形分类
        tr.append("td").append("svg").attr("width", '16').attr("height", '16').append("rect")
            .attr("width", '16').attr("height", '16')
			.attr("fill",function(d){ return color(d.type); });
		//第二列	 分段
		tr.append("td").text(function(d){ return d.type;});

        // 第三列：分段数额.
        tr.append("td").attr("class",'legendgrade')
            .text(function(d){ return d3.format(",")(d.grade);});

        // 第四列：分段占比
        tr.append("td").attr("class",'legendPerc')
            .text(function(d){ return getLegend(d,lD);});	
            
        leg.update = function(nD){//根据直方图数据nD来变
            // 更新附加到行元素的数据。
            var l = svg_3.select("tbody").selectAll("tr")
            .data(nD)
            .transition()
            .duration(1000);

            // 更新第三列.
            l.select(".legendgrade").text(function(d){ return d3.format(",")(d.grade);});

            // 更新第四列.
            l.select(".legendPerc").text(function(d){ return getLegend(d,nD);});        
        }
        
        function getLegend(d,aD){ // Utility function to compute percentage.
            return d3.format("%")(d.grade/d3.sum(aD.map(function(v){ return v.grade; })));
        }
         return leg;
    }

    var tF = ['failed','Pass','medium','excellent'].map(function(d){ 
        return {type:d, grade: d3.sum(fData.map(function(t){ return t.grade[d];}))}; 
    });    
  
    var sF = fData.map(function(d){return [d.course,d.total];});
    var hG = histoGram(sF), //创建histogram.
        pC = pieChart(tF), // 创建pie-chart.
        leg= legend(tF);  // 创建the legend.
        
}
      
</script>
<script>
var gradeData=[
{course:'Chi',grade:{failed:1600,Pass:2786, medium:1319, excellent:249}}
,{course:'Math',grade:{failed:540,Pass:901, medium:412, excellent:674}}
,{course:'Eng',grade:{failed:345,Pass:932, medium:2149, excellent:418}}
,{course:'Phy',grade:{failed:231,Pass:832, medium:1152, excellent:1862}}
,{course:'Chem',grade:{failed:1090,Pass:3400, medium:3304, excellent:948}}
,{course:'Bio',grade:{failed:670,Pass:1619, medium:167, excellent:1063}}
,{course:'PE',grade:{failed:900,Pass:1819, medium:247, excellent:1203}}
,{course:'Geo',grade:{failed:1145,Pass:2498, medium:3852, excellent:942}}
,{course:'His',grade:{failed:78,Pass:797, medium:1849, excellent:1534}}
,{course:'Art',grade:{failed:180,Pass:162, medium:379, excellent:471}}
];

dashboard(gradeData);
</script>
</body>
</html>
