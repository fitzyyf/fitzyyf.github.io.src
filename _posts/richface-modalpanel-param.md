title: "richfaces传递参数给modalPanel并显示modalPanel"
date: 2008-11-04 11:58
tags: richfaces
categories:  
- 编程语言
---
richfaces中使用modalPanel比较不错，界面看起来很炫，但是在一般的应用中一定会打开某个modalPanel的时候会执行下后台的action来刷新modalPanel，这个时候传递参数好像很麻烦，在Google上找了半天，终于在[url]http://www.findjar.com/[/url]网站上找到了解决办法。方法如下：

执行后台action后打开modalPanel：

{% codeblock button lang:html %}
<a4j:commandButton value="打开面板" reRender="table" 
 	action="#{dataTableScrollerBean.takeSelection}" 
	oncomplete="javascript:Richfaces.showModalPanel('panel');"/>
{% endcodeblock %}

传递参数执行action后打开modalPanel（参考[url]http://www.findjar.com/[/url]）：

{% codeblock modalpanel lang:html %}
<rich:modalPanel id="addCardPopup" resizeable="false" width="500">
	<h:panelGrid columns="1" width="100%">
		<h:panelGrid columns="1" width="100%">
			<a:outputPanel id="addToInventory">
				<a:form id="addToInventoryForm">
					<h:panelGrid columns="2" width="100%">
						<h:outputText value="Card: " />
						<h:outputText id="cardName"
							value="#{inventoryHome.card.cardName}" />
						<h:outputText value="Number of copies: " />
						<h:inputText id="add" value="#{inventoryHome.add}" />
						<h:inputHidden id="addCardId" value="#{inventoryHome.addCardId}" />
						<a:commandButton id="addToInventoryButton" value="Add Card"
							action="#{inventoryHome.addCard()}"
							onclick="Richfaces.hideModalPanel('addCardPopup')" />
					</h:panelGrid>
				</a:form>
			</a:outputPanel>
		</h:panelGrid>
		<h:commandButton value="Close"
			onclick="Richfaces.hideModalPanel('addCardPopup')" />
	</h:panelGrid>
</rich:modalPanel>
{% endcodeblock %}

richfaces表格中加入该buttion：

{% codeblock modalpanel in table lang:html %}
<h:column rendered="#{identity.loggedIn}">
	<f:facet name="header">Inventory</f:facet>
	<a:form>
		<a:outputPanel id="addCardPopupPanel">
			<a:commandButton value="Add..." reRender="addCardPopup"
				oncomplete="Richfaces.showModalPanel('addCardPopup', {})">
				<a:actionparam name="card" value="#{card.id}"
					assignTo="#{inventoryHome.addCardId}" />
			</a:commandButton>
		</a:outputPanel>
	</a:form>
</h:column>
{% endcodeblock %}

还有个未尝试：

{% codeblock modalpanel-other lang:html %}
<a:commandButton value="Add..." reRender="addCardPopupGrid"
	ajaxSingle="true" action="#{inventoryHome.setAddCardId(card.id)}"
	oncomplete="Richfaces.showModalPanel('addCardPopup', {})">
</a:commandButton>
{% endcodeblock %}
