<template>
	<div class="calendar">
		<div class="calendarHeader">
			<div class="back">
				<img src="../assets/back.png" height="25" width="25" alt="back">
			</div>
			<div class="title">Calendar</div>
			<div @click='sort' class="sort">
				<img src="../assets/sort.png" height="25" width="25" alt="sort">
			</div>
		</div>
		<div class="body"> 
			<div class="bodyHeader">
				<div class="bodyHeaderContainer">
					<div @click='now' class="month">{{this.month_name[month]}}</div>
					<div class="pageTurn">
						<span @click='prevMonth' class='prevMonth'>
							<img src="../assets/prev.png" height="20" width="20" alt="prevMonth">
						</span>
						<span @click='nextMonth' class='nextMonth'>
							<img src="../assets/next.png" height="20" width="20" alt="nextMonth">
						</span>
					</div>
					<div @click='addEvent' class='add'>
						<img src="../assets/add.png" height="30" width="30" alt="add">
					</div>
				</div>
				<span class="year">{{year}}</span>
			</div>
			<div class="weekDay">
				<ul>
					<li>MON</li>
					<li>TUE</li>
					<li>WED</li>
					<li>THU</li>
					<li>FRI</li>
					<li>SAT</li>
					<li>SUN</li>
				</ul>
			</div>
			<div class="days">
				<ul >
					<li v-for='(item, index) in daysList' class="day">
						<span @click='select(index)' ref="selectChild" :class='{"select": isSelect == index, "mark": todayIndex == index,"disabledDays": disabledDays.indexOf(index) >= 0}'>{{item}}</span>
					</li>
				</ul>
			</div>
		</div>
		<div class="notes">
			<ul>
				<li v-for='(item, index) in eventData' class="note">
					<div ref='colorType' class="colorType"></div>
					<span class="noteDate">{{`${item.year}-${item.month +1 }-${item.date}`}}</span>
					<p class="event">{{item.event}}</p>
					<span @click='removeEvent(item, index)' class="removeEvent">
						<img src="../assets/cancel.png" height="10" width="10"  alt="cancel">
					</span>
				</li>
			</ul>
		</div>
		<div class="inputCantainer" v-show='isBoxShow'>
			<div  class="inputBox">
				<div class="eventDate">{{`${year}-${month +1}-${daysList[isSelect]}`}}</div>
				<input  v-model='eventMessage'  placeholder="日程" class="textBox" type='text' autofocus="autofocus"></input>
				<div class="line"></div>
				<span @click='cancel' class="cancel">取消</span>
				<span @click='ok' class="ok">确定</span>
			</div>
		</div>
	</div>
</template>
<script>
	export default {
		name: 'calendar',
		data () {
			return {
				month: Number,     //当前页的月份
				year: Number,      //当前页的年份
				stratDay: Number,  //每月1号周几
				curEndDay: Number, //每月最后一天周几
				daysList: [],      //每次翻页存放要显示的日期数据
				todayIndex: Number,//今天位于数据中的位置
				isSelect: Number,  //选中的日期位于数据中的位置
				disabledDays: [],  //每月前后不可选中的数据
				eventMessage: '',  //日程信息
				isBoxShow: false,  //日程框显示
				eventData: [],     // 所有日程事件数据
				isSort: false,     //排序状态
				month_name: ["January","Febrary","March","April","May","June","July","Auguest","September","October","November","December"]
			}
		},
		mounted: function () {
			this.init();
		},
		methods: {
			init: function () {
				this.monthAndYear();  //获取年月
				//事例
				this.setStorage([{year: 2019, month: 4, date: 9, event: "同学聚会", inDaysListIndex: 10, color: "#f05b72"}])
				this.eventData = this.getStorage() || [];  //是否有本地数据
				this.refresh();  //每次翻页刷新数据
			},
			monthAndYear: function () {
				var date = new Date();
				this.year = date.getFullYear();
				this.month = date.getMonth();
			},
			refresh: function () {
				var month_olympic = [31,29,31,30,31,30,31,31,30,31,30,31];  // 闰月
				var month_normal = [31,28,31,30,31,30,31,31,30,31,30,31];   //平月
				if(this.year % 4 == 0 && this.year % 100 != 0 || this.year % 400 == 0) {
					this.renderDaysList(month_olympic);  //判断是否是闰月
					
				}else{
					this.renderDaysList(month_normal);
				}
			},
			renderDaysList: function (isMonth_olympic){
				this.computeDates(isMonth_olympic);  //为daysList添加每次翻页存放要显示的日期数据
				this.disabledDay(isMonth_olympic);  //本月前后其它月份不可点击
				this.todayMark();      //标记今天
				this.markEventDate();  //高亮保存的日程
				this.refreshColor();   //为之前添加的日程note添加颜色
			}, 
			computeDates: function(isMonth_olympic) {
				this.daysList = [];  //清空旧数据，为下边刷新新数据准备
				var preTotalDays;  //上月总天数
				 this.stratDay = new Date(this.year, this.month, 1).getDay(); //此月第一天周几
				 this.curEndDay = new Date(this.year, this.month, isMonth_olympic[this.month]).getDay();  //此月最后一天周几
				 
				//解决1月份无法获取上年12月天数
				if(this.month == 0){
					preTotalDays = isMonth_olympic[11];
				}else {
					preTotalDays= isMonth_olympic[this.month - 1];
				}
				//本月前加入数组
				if(this.stratDay != 1){
					if(this.stratDay == 0){
						var stratDay = 7;
						for (var i = preTotalDays; i > preTotalDays - stratDay + 1; i--) {
							this.daysList.unshift(i);

						};
					}else{
						for (var i = preTotalDays; i > preTotalDays - this.stratDay + 1; i--) {
							this.daysList.unshift(i);
						};
					}
				}
				//本月加入数组
				for (var i = 1; i <= isMonth_olympic[this.month]; i++) {
					this.daysList.push(i);
				};
				//本月后加入数组
				if(this.curEndDay > 0){
					for (var i = 1; i < 7 - this.curEndDay + 1; i++) {
						this.daysList.push(i);
					};
				}
			}, //添加日程
			ok: function () {
				this.isBoxShow = false;
				var colors = ['f7acbc','ef5b9c','f05b72','f58f98','f391a9','d71345','8a2e3b','f58220','905a3d','d3c6a6','dbce8f','ffd400','fcf16e','596032','b2d235','a3cf62','1d953f','77ac98','007d65','45b97c','50b7c1','009ad6','008792','585eaa','543044','c77eb5','ea66a6','d3d7d4','6c4c49','72777b','d1c7b7']; //随机颜色
				var color = '#' + colors[Math.floor(Math.random() * 25)];
				//将添加的日程数据存入 eventData
				this.eventData.push({year: this.year, month: this.month, date: this.daysList[this.isSelect], event: this.eventMessage, inDaysListIndex: this.isSelect, color: color});
				//将更新后的eventData存入localStorage
				this.setStorage(this.eventData);
				//高亮此次添加的日程
				this.markEventDate();
				// 添加日程成功后为本次日程note添加颜色
				this.addNoteColor(color);
				this.eventMessage = '';
			},  
			// 删除日程并刷新日程和日期颜色
			removeEvent: function (item, index) {
				this.eventData.splice(index, 1);
				this.setStorage(this.eventData);
				this.markEventDate(); //刷新日期高亮
				this.refreshColor();  //刷新日程color颜色				
			},  // 点击选中
			select: function (index) {
				if(this.disabledDays.indexOf(index) < 0){
					this.isSelect = index;		
				}				
			},  //向上翻页
			prevMonth: function () {
				if(this.month === 0){  //翻页超出重置
					this.month = 11;
					this.year = this.year - 1;
				}else {
					this.month = this.month - 1;
				}
				this.isSelect = null;  //翻页取消选中
				this.refresh();
			},  //向下翻页
			nextMonth: function () {
				if(this.month === 11){  //翻页超出重置
					this.month = 0;
					this.year = this.year + 1;
				}else {
					this.month = this.month + 1;
				}
				this.isSelect = null;
				this.refresh();
			},  // 点击月份返回本月
			now: function () {
				this.isSelect = null;  //翻页取消选中
				this.monthAndYear();   //返回本月
				this.refresh();
				
			},  //弹框确认
			addEvent: function () {
				if(typeof this.isSelect == 'number') {
					this.isBoxShow = true;
				}
			},  //弹框关闭
			cancel: function () {
				this.isBoxShow = false;
				this.eventMessage = '';
			},  //日程排序
			sort: function () {
				var oldArr = [];  //旧日期
				var newArr = [];  //排序后日期
				var str,month,date,sortEventData;
				sortEventData = [];
				//为所有日程日期格式化
				for (var i = 0; i < this.eventData.length; i++) {
					if(this.eventData[i].month < 9){
						month = `0${this.eventData[i].month}`;
					}else{
						month = this.eventData[i].month;
					}
					if(this.eventData[i].date < 10){
						date = `0${this.eventData[i].date}`
					}else{
						date = this.eventData[i].date;
					}
					str = this.eventData[i].year + month + date;
					oldArr.push(parseInt(str, 10));
					newArr.push(parseInt(str, 10));
				};
				//对新日期排序并映射具体数据内容
				newArr.sort(function(a, b){return a - b});
				for(var i = 0; i < oldArr.length; i ++) {
					var index = oldArr.indexOf(newArr[i]);
					sortEventData.push(this.eventData[index]);
				}
				//排序
				if (this.isSort == false) {
					this.isSort = true;
					this.eventData = sortEventData;
					this.refreshColor();
				}else{
					//取消排序
					this.isSort = false;
					this.eventData = this.getStorage();
					this.refreshColor();
				}				
			},
			//高亮今天
			todayMark: function () {
				var date = new Date();
				var year = date.getFullYear();
				var month = date.getMonth();
				var stratDay = new Date(year, month, 1).getDay();
				if(this.year == year && this.month == month){
					this.todayIndex = date.getDate() + stratDay -2;
				}else{
					return this.todayIndex = null;
				}
			}, //把日程数据存入本地
			setStorage: function (eventData) {
				localStorage.removeItem('data');
				localStorage.setItem('data', JSON.stringify(eventData));

			},   //获取本地日程数据
			getStorage: function () {
				if(localStorage.data) {
					return JSON.parse(localStorage.getItem('data'));
				}
			}, //为所有日程note添加颜色
			refreshColor: function () {
				for(let i = 0; i < this.eventData.length; i ++) {
					this.$nextTick(function() {
						this.$refs.colorType[i].style.background = `${this.eventData[i].color}`;
					})
				}
			},  //为本次日程note添加颜色
			addNoteColor: function (color){
				this.$nextTick(function () {
					this.$refs.colorType[this.eventData.length - 1].style.background = `${color}`;
				})
			}
			,//刷新高亮日程
			markEventDate: function () {
				//删除所有日期样式
				for(let i = 0; i < this.daysList.length; i++) {
					this.$nextTick(function () {
						this.$refs.selectChild[i].style.border = `2px solid #fff`;
					})
				}
				//高亮日程
				if(this.eventData.length != 0) {
					for(let i = 0; i < this.eventData.length; i ++) {
						if(this.year === this.eventData[i].year && this.month === this.eventData[i].month){
							this.$nextTick(function () {
								this.$refs.selectChild[this.eventData[i].inDaysListIndex].style.border = `2px solid ${this.eventData[i].color}`;
							})
						}
					}
				}
			}, //添加本月前后其它月份不可点击的数据
			disabledDay: function (isMonth_olympic) {  
				this.disabledDays = [];   //每次翻页清空不可点击的数据
				var stratDay;
				if(this.stratDay != 1){   //此月前不可点击数据存入disabledDays数组
					if(this.stratDay == 0){
						stratDay = 7;
						for(var i = 0; i < stratDay - 1;i++){
							this.disabledDays.push(i);
						}
					}else{
						for(var i= 0; i < this.stratDay - 1;i++){
							this.disabledDays.push(i);
						}
					}
				}
				if(this.curEndDay > 0){  //此月后不可点击数据存入disabledDays数组
					if(this.stratDay == 0){
						stratDay = 7;
						for(var i = stratDay + isMonth_olympic[this.month] - 1; i < this.daysList.length; i++){
							this.disabledDays.push(i);
						}
					}else{
						for(var i = this.stratDay + isMonth_olympic[this.month] - 1; i < this.daysList.length; i++){
							this.disabledDays.push(i);
						}
					}
				}
			}
		}
	}
</script>
<style scoped lang='scss'>
	.calendar{
		-moz-user-select:none;
		-webkit-user-select:none;
		-ms-user-select:none;
		-khtml-user-select:none;
		-o-user-select:none;
		user-select:none;
		
		&::-webkit-scrollbar {
			display: none;
		}
		-ms-overflow-style: none;
		overflow: -moz-scrollbars-none; 

		width: 100%;
		@media only screen and (min-width: 600px) {
			width: 414px;
			height: 650px;
			overflow: auto;
			position: relative;
			left: 50%;
			transform: translateX(-207px);
			border: 1px solid #f1f1f1;
		}
		font-family: STXihei, "Microsoft YaHei";
		.calendarHeader { 
			position: relative;
			width: 100%;
			height: 85px;
			border-radius: 13px 13px 0 0;
			background: -webkit-linear-gradient(left, rgba(244,137,161,1), rgba(238,58,135,1));
			background: -o-linear-gradient(left, rgba(244,137,161,1), rgba(238,58,135,1));
			background: linear-gradient(to right, rgba(244,137,161,1), rgba(238,58,135,1)); 
			.back{
				display: inline-block;
				position: absolute;
				top: 25px;
				left: 30px;
			}
			.title{
				line-height: 85px;
				text-align: center;
				font-size: 20px;
				color: #fff;
			}
			.sort{
				display: inline-block;
				position: absolute;
				top: 25px;
				right: 30px;
			}
		}
		.body {
			margin-top: -10px;
			width: 100%;
			border-radius: 13px 13px 0 0;
			background: #fff;
			.bodyHeader {
				width: 100%;
				margin-bottom: 50px;
				overflow: hidden;
				.bodyHeaderContainer {
					margin-left: 30px;
					margin-top: 50px;
					overflow: hidden;
					.month {
						float: left;
						line-height: 35px;
						font-size: 35px;
					}
					.pageTurn {
						float: left;
						margin-left: 15px;
						.prevMonth {
							display: block;
							width: 20px;
							height: 20px;
						}
						.nextMonth {
							display: block;
							width: 20px;
							height: 20px;
						}
					}
					.add {
						float: right;
						width: 40px;
						height: 40px;
						border-radius: 50%;
						margin-right: 30px;
						text-align: center;
						background: rgba(238,58,135,1);
						img {
							display: inline-block;
							position: relative;
							top: 5px;
						}
					}
				}
				.year {
					display: block;
					line-height: 20px;
					margin-left: 50px;
					font-size: 20px;
				}
			}
			.weekDay {
				width: 100%;
				ul {
					display: -webkit-flex;
					display: flex;
					flex-flow: row nowrap;
					margin: 10px 30px;
					li{
						flex-grow: 1;
						display: inline-block;
						width: 14%;
						height: 20px;
						line-height: 20px;
						text-align: center;
						font-size: 15px;
						color: #8a8a8a;
					}
				}
			}
			.days {
				width: 100%;
				position: relative;
				ul {
					display: -webkit-flex;
					display: flex;
					flex-flow: row wrap;
					margin: 15px 30px;
					li{
						flex-grow: 1;
						display: inline-block;
						width: 13%;
						padding: 5px 0;
						font-size: 15px;
						color: #8a8a8a;
						span{
							display: block;
							width: 30px;
							height: 30px;
							margin: 0 auto;
							border: 2px solid #fff;
							border-radius: 50%;
							line-height: 32px;
							text-align: center;
						}
						.mark{
							border: 2px solid #fff;
							color: #fff;
							background: -webkit-linear-gradient(left, rgba(244,137,161,1), rgba(238,58,135,1));
							background: -o-linear-gradient(left, rgba(244,137,161,1), rgba(238,58,135,1));
							background: linear-gradient(to right, rgba(244,137,161,1), rgba(238,58,135,1)); 
						}
						.select{
							border: 2px solid #8a8a8a;
							box-shadow: 0 1px 2px 4px #dbdbdb;
						}
						.disabledDays{
							color: #dbdbdb;
						}
						
					}
				}
			}
		}
		.notes{
			width: 100%;
			padding: 20px 0;
			margin-top: -15px;
			background: #f1f1f1;
			.note{
				position: relative;
				width: 91%;
				height: 50px;
				margin: 10px auto;
				border-radius: 13px;
				font-size: 12px;
				background: #fff;
				color: #8a8a8a;
				.colorType{
					position: absolute;
					top: 20px;
					left: 15px;
					width: 10px;
					height: 10px;
					border-radius: 5px;
				}
				.noteDate{
					position: absolute;
					top: 8px;
					left: 40px;
				}
				.event{
					position: absolute;
					bottom: 6px;
					left: 40px;
					width: 80%;
					height: 18px;
					overflow: hidden;
				}
				.removeEvent{
					width: 10px;
					height: 10px;
					position: absolute;
					right: 10px;
					top: 10px;
				}
			}
		}
		.inputCantainer{
			position: fixed;
			left: 50%;
			top: 50%;
			transform: translate3d(-100px,0,0);
			width: 200px;
			border: 1px solid #000;
			border-radius: 13px;
			background: #fff;
			color: #8a8a8a;
			.inputBox{
				width: 180px;
				margin: 0 auto;
				.eventDate{
					padding-top: 15px;
					font-size: 14px;
					line-height: 14px;
				}
				.textBox{
					width: 100%;
					height: 25px;
					margin-top: 15px;
					font-size: 14px;
					outline: none;
				}
				.line{
					width: 100%;
					border-top: 1px solid #000;
				}
				span{
					display: inline-block;
					width: 40px;
					height: 30px;
					margin-top: 10px;
					text-align: center;
					line-height: 30px;
				}
				.cancel{
					float: left;
					margin-left: 30px;
				}
				.ok{
					float: right;
					margin-right: 30px;
				}
			}
		}
	}
</style>
