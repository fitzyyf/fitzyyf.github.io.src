title: "flex设计表格的复杂表头(类似报表的斜线表头)"
date: 2009-05-06 16:02
tags: 
- flex
categories: 
- 开发笔记
---

前段时间，项目中开始使用flex了。碰到一个难题，flex的dagagrid和advancedDataGrid的表头没有斜线(好像没有什么控件会考虑这个)

{% img /images/blog/nsbdqkb.jpg %}

而自己对flex只是熟悉控件的使用，搜索无果，幸好公司的技术顾问是这方面的高手吧。就让他帮忙给搞好了。感谢。。

主要文件

{% codeblock flex-source lang:xml %}
<?xml version="1.0" encoding="utf-8"?>
<mx:Canvas xmlns:mx="http://www.adobe.com/2006/mxml" 
	creationComplete="init()">

	<mx:Script>

		<![CDATA[
			import mx.events.ResizeEvent;
			import mx.controls.Label;

			private const leftText:String = "月份";

			private const rightText:String = "户型";

			private var leftLabel:Label;

			private var rightLabel:Label;

			private function init():void{
				addLabel();
				drawLine();				
				this.addEventListener(ResizeEvent.RESIZE, onResize);
			}

			private function onResize(e:ResizeEvent):void{
				resetLabel();
				drawLine();
			}

			private function drawLine():void{
				var g:Graphics = this.graphics;
				g.clear();
				g.lineStyle(0.5, 0xB7BABC);
				g.moveTo(0, 0);
				g.lineTo(this.width, this.height);
			}

			private function addLabel():void{
				leftLabel = new Label();
				leftLabel.text = leftText;
				addChild(leftLabel);

				rightLabel = new Label();
				rightLabel.text = rightText;
				addChild(rightLabel);
				callLater(resetLabel);
			}

			private function resetLabel():void{
				leftLabel.x = 5;
				leftLabel.y = this.height - leftLabel.height - 5;
				rightLabel.x = this.width - rightLabel.width;
				rightLabel.y = 10;
			}
		]]>
	</mx:Script>	
</mx:Canvas>
{% endcodeblock %}

使用

{% codeblock use lang:xml %}
<mx:AdvancedDataGridColumn dataField="yf" headerText="月份" headerWordWrap="true"
					headerRenderer="MyRenderer"/>
{% endcodeblock %}

效果图:

{% img /images/blog/flex-datagrid.png %}


