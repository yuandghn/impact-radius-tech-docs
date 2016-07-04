`文档处于不断地更新和完善的过程中，我们会把各个Advertiser接入过程中遇到的各种问题记录下来并分享给所有的Advertiser。`  
`因IR文档系统升级，如果您发现文中某些链接已经失效请告知我们，也可到IR新的Help Center(https://help.impactradius.com/hc/en-us)中查看或搜索相关的文档。`

###角色
[Extrabux](http://www.extrabux.com/)是Media Partner，电商网站/转运公司是Advertiser，[Impact Radius](http://www.impactradius.com/)(以下简称IR)是广告联盟。Extrabux和Advertiser通过IR系统联系在一起。
      
###业务
概括来说就是Extrabux为Advertiser带去流量和用户，用户在Advertiser上产生一个转化行为后，Advertiser支付一笔佣金给Extrabux，然后Extrabux再将佣金的绝大部分返还给用户。  

- ######注册返利
Extrabux每成功推荐一个用户在Advertiser上注册，Advertiser返还给Extrabux一笔固定金额的佣金，Extrabux再将佣金的绝大部分返还给用户。
- ######充值返利
Extrabux引导用户去Advertiser上充值，Advertiser将用户的充值金额的一定比例作为佣金返还给Extrabux，Extrabux再将佣金的绝大部分返还给用户。
- ######下单返利
Extrabux引导用户去Advertiser上消费，用户在Advertiser上产生消费行为后，Advertiser将用户的订单金额[^1]的一定比例[^2]作为佣金返还给Extrabux，Extrabux再将佣金的绝大部分返还给用户。

IR在Extrabux和Advertiser中间起到桥梁的作用。

佣金的形式、数额由Extrabux和Advertiser约定好并在IR进行配置，双方不产生任何直接的行为交互和资金流动。它们各自做自己的事情并和IR打交道，由IR作为媒介来串起整个流程。


###流程细节
1. 用户在Extrabux上登录
2. 点击目标Advertiser的Ad
3. Extrabux调用IR提供的此Ad的url并传入当前用户的标识信息
>
Extrabux和IR之间的交互至此结束，Advertiser无须关心以上流程。下面进入IR和Advertiser之间的交互。

4. IR引导用户进入此Ad url对应的Advertiser提供的入口页面(我们现在推荐做`全局拦截`而不是定义一个具体的入口页面，详细原因请看页脚注释[^3])，并传入此次Ad click event的惟一标识和来源标识，即clickid[^4]和irpid[^5]。Extrabux的irpid是`126888`。   
Advertiser须将这两个参数[^6]保存到cookie里并设置一定的有效期(一般半个月即可)，因为不知道用户成功转化需要多长时间。
>
已经接入到IR里的商家使用的参数是`irmpname`而非`irpid`，我们现在推荐使用`irpid`，原因是我们发现`irmpname`的值是可以被人为修改的，而`irpid`的值是固定不变且惟一的。后面如有需要，我们会协助已接入的商家使用新的`irpid`参数。

5. 用户成功转化后，Advertiser回传数据给IR。建议Advertiser将clickid、irpid和订单/运单信息关联起来，方便以后自己做查询统计；并对回传成功的记录打上标记，以便失败后重传。  
回传的数据内容主要包括：CampaignId、ActionTrackerId、ClickId、Oid、EventDate、Amount、Currency等[^7]。  
以下是各个参数的说明。  
	- `CampaignId`：固定值，每个ActionTracker均使用相同的CampaignId，在对ActionTracker做测试时就可以看到。
	- `ActionTrackerId`：固定值，每个ActionTracker的id都不一样，在对ActionTracker做测试时就可以看到。
	- `ClickId`：第4步中IR传入的参数
	- `Oid`：即Order Id，能惟一标识此次交易的id，建议直接使用订单/运单id，一来方便用户在Extrabux中查看[^8]，二来便于以后数据有问题时对账。
	- `EventDate`：交易完成的时间点，如订单/运单成功支付的时间点。格式：`dd-MMM-yyyy hh:mm:ss z`，如`21-Oct-2015 18:25:59 HKT`[^9]。小时请使用`24小时制`，因为在有些编程语言中`mm`表示的是12小时制，而`MM`表示的却是24小时制。此外，`z`在不同的编程语言中代表的意思也可能不一样，请换成你所使用的编程语言中能格式化成三个字母的用以代表时区的标识。
	- `Amount`：订单/运单金额。Advertiser可根据自身业务需求决定是否剔除优惠券、积分等内容。
	- `Currency`：可选项。货币的缩写。如果Advertiser和用户之间的交易不是以美元来结算的，请设置此属性，如人民币的货币缩写为CNY[^10]。因为IR默认是以美元来结算，设置Currency后，IR会自动根据当前汇率将Amount换算成美元。
7. IR接收到数据后，出Reports给Advertiser和Extrabux看。数据的锁定期默认是15天(可修改)。在锁定期内，Advertiser可对数据进行修正、Approve、Reverse等；锁定期一过，意味着Advertiser认可交易数据，IR随后会将佣金转给Extrabux。
8. Extraubx收到佣金后再与用户进行结算。  

到此，流程完毕。

###数据回传方式
不同的数据回传方式是在创建Action Tracker时的Tracking Method属性中指定的，创建后无法更改。Action Tracker可以按照字面意思自己理解，Advertiser可创建N多个Action Tracker来trace各种不同的action，如注册、充值、下单等。  

######以下是几种常用的Tracking Method：  
- Pixel  
这是一种在交易完成时实时[^11]回传数据的方式。它通过在网页里嵌入JavaScript代码的方式来回传数据。一般不适用于交易完成后的状态可能会发生变化的情况，如订单取消等。  
更多信息请参考文档[JavaScript Conversion Code](https://help.impactradius.com/hc/en-us/articles/203769585-Javascript-Conversion-Code)。

- Batch FTP  
这是一种通过上传数据文件到IR的FTP服务器来回传数据的方式。IR接受两种格式的数据文件：XML和CSV。推荐使用可读性更强的XML格式。各个编程语言都有FTP client可以使用。  
创建此类型的Action Tracker时点击`Email FTP Username and Password`按钮， IR会把FTP服务器的用户名/密码发到Advertiser的注册邮箱里。  
那何时触发FTP上传动作呢？以转运公司为例，一般可选择在交易状态已基本稳定的时间点进行，如包裹出库时立即上传此运单的数据，或者定时触发，如一天上传一次，这时数据文件里一般会包含多笔运单的数据。  
XML格式数据样例：
######Conversion Request
		<?xml version="1.0" encoding="UTF-8"?>
		<ConversionRequests xmlns="http://ws.impactradius.com/ws/definitions">
    		<ConversionRequest>
        		<CampaignId>3141</CampaignId>
        		<ActionTrackerId>8232</ActionTrackerId>
        		<ClickId>3AiWUu2QnUtyxPe3eRX0LVC9UkXWJ6UlqxHu2U0</ClickId>
        		<Oid>OT-201510191230</Oid>
        		<Amount>99</Amount>
        		<EventDate>19-Oct-2015 12:36:30 HKT</EventDate>
        		<Currency>CNY</Currency>        
    		</ConversionRequest>
    		<ConversionRequest>
        		...
    		</ConversionRequest>  
    		<ConversionRequest>
        		...
    		</ConversionRequest> 
    		...... 
		</ConversionRequests>
######Items Conversion Request
		<?xml version="1.0" encoding="UTF-8"?>
		<ItemConversionRequests xmlns="http://ws.impactradius.com/ws/definitions">
    		<ItemConversionRequest>
        		<CampaignId>3141</CampaignId>
        		<ActionTrackerId>8232</ActionTrackerId>
        		<ClickId>3AiWUu2QnUtyxPe3eRX0LVC9UkXWJ6UlqxHu2U0</ClickId>
        		<Oid>OT-201510191230</Oid>
        		<CustomerId>40111728</CustomerId>
        		<EventDate>19-Oct-2015 12:36:30 HKT</EventDate>
        		<Currency>CNY</Currency>
        		<Items>
            		<Item>
                		<Sku>sku-40892</Sku>
                		<Category>cat-1002</Category>
                		<Quantity>1</Quantity>
                		<Amount>29.00</Amount>
            		</Item>
            		<Item>
             	   		<Sku>sku-1373</Sku>
                		<Category>cat-3002</Category>
                		<Quantity>2</Quantity>
                		<Amount>36</Amount>
            		</Item>
        		</Items>
    		</ItemConversionRequest>
    		<ItemConversionRequest>
        		...
    		</ItemConversionRequest>        
    		<ItemConversionRequest>    
        		...
    		</ItemConversionRequest>
    		......    
		</ItemConversionRequests>
`重要提示1`：The Item data needs to be ordered first by OrderId, then Item SKU. If records are out of order, the items will be viewed as duplicate orders and only the first item will count.  
`重要提示2`：请务必将样例中的CampaignId和ActionTrackerId替换成你们自己的。  
`重要提示3`：请尽量不要上传重复数据。  
Items Conversion Request一般用于电商网站；Conversion Request则一般用于转运公司。当然，如果Advertiser想把一个运单也当作一件Item的话，用Items Conversion Request也无妨。  
数据文件上传成功后，Advertiser的注册邮箱里会收到一封IR发出的标题为`Data Submission Received`的邮件，里面是对此次上传的数据的处理结果。Batch Details中Failure为False时数据才算上传成功。  
数据文件默认是上传到IR的FTP服务器的根目录下的，Advertiser无须考虑创建不同的文件夹来管理上传的数据文件，IR也没说允许这么做。如果您使用的是可视化上传工具，如FileZilla之类的，在上传成功后应该是看不到刚才所上传的文件的，这是正常现象，IR有其自己的处理方式，无需担忧。  
更多信息请参考文档：[Batch FTP Tracking Method](http://support.impactradius.com/display/ADVERTISER/Batch+FTP+Tracking+Method)、[Advertiser FTP Conversion Reporting Method](http://support.impactradius.com/display/ADVERTISER/Guide+-+Tracking+Integration#Guide-TrackingIntegration-AdvertiserFTPConversionReportingMethodforOnlineActions)
- Web Services  
这里的Web Services不是通常意义上的Web Service，其实它就是个HTTP REST API。各个编程语言都有HTTP Client可以使用。  
请详细阅读文档[Web Services Tracking](http://support.impactradius.com/display/ADVERTISER/Web+Services+Tracking)中关于`authenticate`的内容。AccountSid和authentication token可以从[这里](https://member.impactradius.com/secure/advertiser/accountSettings/techintegration/adv-wsapi-flow.ihtml?execution=e2s1)获得。  
更多信息请参考文档：[Web Services Tracking](http://support.impactradius.com/display/ADVERTISER/Web+Services+Tracking)、[API-Conversions](http://dev.impactradius.com/display/api/Conversions)、[Response Formats](http://dev.impactradius.com/display/api/Getting+Started#GettingStarted-responseFormatsResponseFormats)      
`纠错`：在文档[Web Services Tracking](http://support.impactradius.com/display/ADVERTISER/Web+Services+Tracking)中Conversions的api接口    
		
		https://api.impactradius.com/2010-09-01/Advertisers/{AccountSid}/Conversions
是错误的。正确的应该是	
  
 	  	https://api.impactradius.com/Advertisers/{AccountSid}/Conversions 
或  

  		https://{AccountSid}:{AuthToken}@api.impactradius.com/Advertisers/{AccountSid}/Conversions  
另外，IR默认返回的是XML格式的数据，如果想让它返回JSON格式的数据，请在url后面加上.json的扩展名，例如  

		https://api.impactradius.com/Advertisers/{AccountSid}/Conversions.json
调用Web Services API成功后类似的返回信息：  
XML格式：

		<ImpactRadiusResponse>
 			<Status>QUEUED</Status>
 			<QueuedUri>/Advertisers/{AccountSid}/BatchSummaries/XysrtIBGGA</QueuedUri> 
		</ImpactRadiusResponse>
JSON格式：  

		{
 			"Status": "QUEUED",
 			"QueuedUri": "/Advertisers/IRCyVFnPHHra3zrNyjz3RTkPoRtri6zSXA/QueuedStatus/wer123456.json"
		}

`注意`：这里的`QUEUED`仅仅表示IR接收到了你回传的数据，但是数据是否Valid、是否会被IR所接受仍然不知道，只有等IR处理完成(非实时)后才知晓。		
失败时类似的返回信息：  
XML格式：

		<ImpactRadiusResponse>
 			<Status>ERROR</Status>
 			<Message>Validation Failed</Message>
 			<Errors>
  				<Error>
   					<Field>CampaignId</Field>
   					<Message>Invalid value xyz</Code>
  				</Error>
 			</Errors>        
		</ImpactRadiusResponse>
JSON格式：  

		{ 
  			"Status": "ERROR",
  			"Message": "Validation Failed",
  			"Errors":  [
         		{
           			"Field": "AccountId",
           			"Message": "Not a numeric value" 
         		}  
   			]   
		}

###Action Tracker  
为各种不同的action定义相应的tracker。  
创建Action Tracker请到[这里](https://member.impactradius.com/secure/advertiser/campaign/actiontracker/view-actiontracker-flow.ihtml?execution=e1s1)。此页面会列出所有已经创建好的的Action Tracker，欲创建新的Action Tracker请点击`Create Action Tracker`按钮。  
前期在Advertiser不熟悉IR的情况下可由Extrabux协助创建Action Tracker，但我们希望后面Advertiser能够自己来掌控Action Tracker，毕竟这是IR里经常用到的功能。  

下面我们重点说一下Action Tracker的测试。`在开始测试之前请Advertiser确保你们的开发人员已经拿到了IR系统的账号`。
######Action Tracker的测试  
点击要测试的Action Tracker的`Actions`按钮，然后选择`Test`。  
如果您测试的是Pixel类型的Action Tracker，请先选择`Tracking Code`来获取要埋入到页面里的JavaScript代码，然后再点击页面下方的`Continue To Testing`即可进入到测试页面。下面是一段JavaScript脚本样例，请认真对待里面的注释内容。

	<!-- Impact Radius Tracking Code.
	Removal or modification of this code will disrupt marketing activities. This code is property of Impact Radius, please do not remove or modify without first contacting Impact Radius Technical Services.
	-->
	<script type="text/javascript" src="//d33wwcok8lortz.cloudfront.net/js/3141/8324/irv3.js"></script>
	<script type="text/javascript">
		// required advertiser supplied values
		irEvent.setOrderId("Your Order Id here");
		
		// At least one item is necessary.  The parameters are
		// cat: A category, for example "electronics".  Required
		// sku: A unique product identifier, or Storage Keeping Unit
		// amt: The total sale amount for this line item.  Required
		// qty: The quantity of the line item.  Required.
		// 一般来说irEvent.addItem(...)方法是放在一个for each循环中的,因为订单中一般都有多个商品,每个商品都需要addItem一下.
  		irEvent.addItem("electronics", "220-2300", "112000.00", "56");  // 56 identical gizmos at 2000.00 each
  		irEvent.setCurrency("CNY"); //可选项，请参考前面对Currency字段的描述。

 		irEvent.fire();
	</script>
细心的同学可能注意到了，这里面回传的参数与我们在流程细节里描述的不一致，根本就没有CampaignId、ActionTrackerId、ClickId和EventDate等参数。是的，在Pixel方式里的确不用显式的传入这些参数，IR会通过其他形式自己获取到。比如，页面加载的"/js/3141/8324/irv3.js"文件里就包含了CampaignId和ActionTrackerId；ClickId在传给你的入口页面之前IR就已经以其他方式保存到自己的cookie里了，当irEvent.fire()的时候就会带过去，同时呢，EventDate也有了。  

如果你想更多的了解这段JavaScript代码背后是怎么工作的，不妨把d33wwcok8lortz.cloudfront.net/js/3141/8324/irv3.js下载下来看下其源码就明白了。可以重点看下addItem()、fire()、getQueryString()等方法。  


测试流程大同小异，我们以Web Service方式为例：   
![web-service-test-page](http://7xnrpy.com1.z0.glb.clouddn.com/web-service-test-page-new.png)
你需要的`CampaignId`、`ActionTrackerId`、`ClickId`都有了。  
Landing Page URL就是我们上面提到的入口页面。这里有个小技巧。当技术人员在本地完成所有的开发工作后，可直接在测试页面对本地项目进行测试，无须部署到线上之后再测，这样可以避免因为一些小问题而频繁的上线。请把Landing Page URL中的域名改成你本地的地址，如127.0.0.1之类的，irpid参数的值可直接替换成`126888`，clickid参数的值IR会自动替换成上面的Test Key。点击`Start Test in New Window`按钮后启动测试，页面里随之会出现一个loading条，IR在等待你回传数据给它。  
`注意事项`：当你使用FTP或Web Service的方式来回传数据时，因为EventDate是显式设置的，不像在Pixel方式里是由IR隐式获得的，所以要特别注意它的值。EventDate的限制条件：`必须是在Click Date之后且不能是一个未来的时间`。Click Date是IR产生clickid的那个时间点，也就是你进入到上图所示的页面的那个时间点。假设这个时间点大约是"19-Oct-2015 12:36:30 HKT"，回传动作发生的时间点大约是"19-Oct-2015 12:42:09 HKT"，那在你回传的数据里EventDate的值必须在这两个时间点之间。  

为了方便开发理解，我使用了Chrome浏览器的Postman插件来回传数据，截图在[这里](http://7xnrpy.com1.z0.glb.clouddn.com/web-service-back-data-to-ir.png)。  
当IR收到回传的数据后，测试页面会进入到`Complete a Conversion`，如下图所示![](http://7xnrpy.com1.z0.glb.clouddn.com/web-service-data-view.png)  
在此页面中校验下IR接收到的数据和你回传的是否一致，如果一致，请挨个`Correct`，然后点击页面下方的`Validate`(也有可能是`Activate Action Tracker`)按钮来激活此Action Tracker。回到列表页面后可以发现此Action Tracker的`Test Status`已经变成`Successful`了。  

####`请一定要记得激活Action Tracker，不然无法进行后续的流程`

Action Tracker的测试到此完成。下面会进入到创建Ad和设置Insertion Order，此部分可先由Extrabux代为操作，但我们仍是希望Advertiser在对IR有一定了解后自己去掌控这些事情。

###Ads
一个Ad创建好后Advertiser可以将它发送给一个或多个或所有的Media Partner[^12]，每个Media Parnter得到的Ad url都不一样。入口页面的url也是在Ad里设置的。

###Insertion Orders
An insertion order is an agreement between a Media Partner and an Advertiser that governs the relationship between the two parties. The Insertion Order contains the terms of the agreement such as payout rates, cookie duration, lead caps, action validation period, return policies, call center hours and all other terms that may be relevant to the partnership. 更多信息请参考[Insertion-Orders](https://help.impactradius.com/hc/en-us/articles/210895837-Insertion-Orders)  
设置Media Partner和Advertiser之间的协议条款。包含但不限于以下几点：返利是百分比还是固定的金额，百分比是多少，固定金额又是多少；从一个Ad click产生到最终完成交易的最长时间间隔；action的锁定期，默认就是上面说过的15天；同一个clickid产生的前几笔交易是有效(返利)的，默认是前3笔[^13]。  
Advertiser可就同一项业务与不同的Media Partner签订不同的协议内容。Advertiser将某个Insertion Order发送给特定的Media Partner，Media Partner接受后双方建立起联系。

###全了
Ads和Insertion Order设置好后，整个流程就通了。  
简单回顾一下。用户在Extrabux上点一个广告(Ad)，Extrabux(或者是其他的Media Partner)知道要将用户导向到哪个Ad url，IR记录下这个Ad click event然后再将用户导向Advertiser提供的对应的入口页面，Advertiser记录下IR传入的参数，用户在Advertiser的网站上成功转化，Advertiser回传数据给IR，IR收到后根据Insertion Order计算佣金然后给到Extrabux。Extrabux再将佣金的绝大部分返还给用户。

###线上模拟测试
这里的线上指的是`生产环境`。  
做线上模拟测试的必要性是因为本地和线上是两个完全不同的环境，本地能跑通的程序到了线上未必也能跑通，如果中间哪个环节存在问题，那么提前暴露出来并解决掉要比正式提供给用户使用后再回来修复所带来的影响要小的多。做线上模拟测试的困难程度对各个Advertiser来说可能不一样，但这个步骤不能省。    
Action Tracker在本地测试通过后由Advertiser部署程序到线上，Angie将对应的Ad url给到Advertiser的技术人员，然后双方可各自进行线上模拟测试。线上模拟测试会省略掉Extrabux到IR的部分，直接从访问Ad url开始。对于Advertiser是转运公司的情况，还需要转运公司能模拟入库、支付、出库等动作。  
IR接收到数据后一般20分钟内就可以在[Pending Actions](https://member.impactradius.com/secure/advertiser/actions/open/pending-actions-flow.ihtml?execution=e12s1)里看到，而在[Reports](https://member.impactradius.com/secure/advertiser/Adv_Campaign_Dashboard/r11/report/viewReport.report?handle=adv_generation_foundation_campaign_dashboard)里看到至少需要2+小时。
  
如果20分钟后还没在Pending Actions里看到回传的数据，那极有可能你回传给IR的数据是Invalid的，IR不予处理。这时，要怎样才能看到IR对此次回传动作的处理结果呢？这要根据你所使用的Tracking Method来定。  
1. Batch FTP  
   请前去注册邮箱里查看标题为`Data Submission Received`的邮件，里面有Batch Summaries的链接，打开后查看Batch Details所指向的内容。  
2. Web Services  
   
######`注意事项`
在做线上模拟测试前，请Angie和Advertiser的技术人员再三确认以下几点：  
1. Action Tracker是否已Validated  
2. Ad的landing page是否设置正确  
3. Angie将Ad url给到Advertiser的技术人员

#####`重要提示`
####线上模拟测试完成后，请Advertiser轻易不要改动相关的代码和逻辑，如确实有此需要，那么在项目部署到生产环境后请自行做一次`线上模拟测试`来保证功能正常。  


######Pending Actions
在action的锁定期内，Advertiser可以对其进行数据修正、Approve、Reverse等操作。对于线上模拟测试的数据，可以在Reverse中选择`Test Action`。  
如果有大量数据需要Reverse可以选择调用Web Services接口或上传数据文件到FTP服务器的方式来批量操作，详情请查看[这里](http://support.impactradius.com/display/ADVERTISER/Understanding+Batch+Reporting+of+Conversion+Returns)。更多关于Pending Actions的文档请查看[FAQs-Pending Actions](http://support.impactradius.com/display/ADVERTISER/FAQs+-+Pending+Actions)

###小问题集锦
Advertisers在实践中会遇到各种各样的小问题，在文档上方的流程细节中无法一一覆盖到，所以我们以问题集锦的方式列出。  
1. 在对Action Tracker测试时，数据已经回传给了IR，但是在Action Trackers列表页面中对应的Test Status却是`System Invalidated`。  
原因：可能是因为你回传的ActionTrackerId和当前测试的action tracker的id不一致造成的。   
2. 在做线上模拟测试时，数据明明已经回传给了IR，过了一段时间后却仍然在Pending Actions里看不到。  
原因：极有可能是EventDate的问题。这里我们再重申一下EventDate的限制条件，它`必须是在Click Date之后且不能是一个未来的时间`。Click Date是IR产生clickid的那个时间点，正常情况下EventDate肯定是在Click Date之后的，但是怕就怕在对EventDate做格式化的时候格错。格式化字符串`dd-MMM-yyyy hh:mm:ss z`是一个整体，不能先把EventDate格式化成`dd-MMM-yyyy hh:mm:ss`后再加上所在的timezone的缩写。如果你使用的编程语言本身(如C#，但C#有第三方库可以做到)不支持格式化出包含timezone缩写的形式，那折衷方案是先把EventDate格式化成UTC时间的字符串然后再拼接上` GMT`(注意GMT前有个空格)来表示使用的时区是GMT。  


[^1]:订单金额里可能包含优惠券、积分等内容，Advertiser可根据自身的业务需求来决定是否剔除它们。
[^2]:也可以是固定金额的佣金。
[^3]:这里虽然说的是入口页面，但其实只要有个后台就行了，不需要真的有对应的页面。因为它只要能接收clickid和irpid参数，然后写到cookie里，然后直接302重定向到首页或其他页面就行了。而且强烈建议在这个后台做302重定向，不然一长串的参数在地址栏里确实不好看。它的职责就是接收IR传入的参数，参数处理完后就重定向到想去的页面。那如何重定向到目标页面呢？Advertiser可以为入口页面额外定义一个target(或redirect_url之类的)参数，当处理完IR传入的参数后再将用户重定向到这个参数所代表的页面。但是，我们更推荐另外一种方式，就是对clickid和irpid参数做`全局拦截`，即无论用户进入的是哪个页面，只要url后面带有这两个参数就对它们进行处理。相比额外加一个target参数的方式，全局拦截可以有效降低IR配置的复杂度，无须每加一个promotion page就需要设置一下对应的target参数值。  
[^4]:每次Ad click都会产生一个惟一的clickid，它是字母、数字和一些符号的组合。
[^5]:Impact Radius Media Partner Id的缩写。
[^6]:IR内置的其他参数和Advertiser可自定义的参数请参考[Campaign Settings -> Online Traffic Settings -> Show advanced settings -> Query String Parameters](https://member.impactradius.com/secure/advertiser/campaign/technicalintegration-flow.ihtml?execution=e1s1)
[^7]:更多的可选项请参考[文档](http://support.impactradius.com/display/ADVERTISER/Batch+FTP+Tracking+Method#BatchFTPTrackingMethod-Fields)
[^8]:是的。Advertiser回传给IR的数据IR同样也会给到Extrabux，Extrabux再显示给用户看。真实有效的Order Id可以让用户很方便的和Advertiser里的订单/运单对应起来。
[^9]:时区缩写请参考[time_zone_abbreviations](https://en.wikipedia.org/wiki/List_of_time_zone_abbreviations)。如果你们系统使用的是中国标准时间，请不要使用CST来表示，因为CST可表示多个时区，而且在IR里也的确不用于表示Chinese Standard Time。请改用HKT(Hong Kong Time)。
[^10]:货币缩写请参考[currency-iso](http://www.currency-iso.org/dam/downloads/lists/list_one.xml)
[^11]:Batch FTP和Web Services这两种tracking method是延后回传数据。
[^12]:体会到irpid参数的用处了吧
[^13]:尽管如此，Extrabux和Advertiser最好还是引导用户每次下单前都从Extrabux点过去，理想状况是一个clickid对应一笔交易。





