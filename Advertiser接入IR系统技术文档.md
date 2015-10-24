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
Extrabux和IR之间的交互至此结束，Advertiser无须关心以上流程。下面进入IR和Advertiser之间的交互

4. IR引导用户进入此Ad url对应的Advertiser提供的入口页面[^3]，并传入此次Ad click event的惟一标识和来源标识，即clickid[^4]和irmpname[^5]。  
Extrabux的irmpname是`Extrabux Shanghai`.  
Advertiser须将这两个参数[^6]保存到cookie里并设置一定的有效期(一般半个月即可)，因为不知道用户成功转化需要多长时间。

5. 用户成功转化后，Advertiser回传数据给IR。建议Advertiser将clickid、irmpname和订单/运单信息关联起来，方便以后查询统计;并对回传成功的记录打上标记，以便失败后重传。  
回传的数据内容主要包括：CampaignId、ActionTrackerId、ClickId、Oid、EventDate、Amount、Currency等[^7]。  
以下是各个参数的说明。  
	- `CampaignId`：固定值，每个ActionTracker均使用相同的CampaignId，在对ActionTracker做测试时就可以看到。
	- `ActionTrackerId`：固定值，每个ActionTracker的id都不一样，在对ActionTracker做测试时就可以看到。
	- `ClickId`：第4步中IR传入的参数
	- `Oid`：即Order Id，能惟一标识此次交易的id，建议直接使用订单/运单id，一来方便用户在Extrabux中查看[^8]，二来便于以后数据有问题时对账。
	- `EventDate`：交易完成的时间点，如订单/运单成功支付的时间点。格式：`dd-MMM-yyyy hh:mm:ss z`，如21-Oct-2015 10:25:59 HKT[^9]
	- `Amount`：订单/运单金额。Advertiser可根据自身业务需求决定是否剔除优惠券、积分等内容。
	- `Currency`：可选项。货币的缩写。如果Advertiser和用户之间的交易不是以美元来结算的，请设置此属性，如人民币的货币缩写为CNY[^10]。因为IR默认是以美元来结算，设置Currency后，IR会自动根据当前汇率将Amount换算成美元。
7. IR接收到数据后，出Reports给Advertiser和Extrabux看。数据的锁定期默认是15天(可修改)。在锁定期内，Advertiser可对数据进行修正操作，如修改和撤消；锁定期一过，意味着Advertiser认可交易数据，IR随后会将佣金转给Extrabux。
8. Extraubx收到佣金后再与用户进行结算。  

到此，流程完毕。

###数据回传方式
不同的数据回传方式是在创建Action Tracker时的Tracking Method属性中指定的，创建后无法更改。Action Tracker可以按照字面意思自己理解，Advertiser可创建N多个Action Tracker来trace各种不同的action，如注册、充值、下单等。  

######以下是几种常用的Tracking Method：  
- Pixel  
这是一种在交易完成时实时[^11]回传数据的方式。它通过在网页里嵌入JavaScript代码的方式来回传数据。一般不适用于交易完成后的状态可能会发生变化的情况，如订单取消等。  
更多信息请参考文档[JavaScript Conversion Code](http://support.impactradius.com/display/ADVERTISER/JavaScript+Conversion+Code)。

- Batch FTP  
这是一种通过上传数据文件到IR的FTP服务器来回传数据的方式。IR接受两种格式的数据文件：XML和CSV。推荐使用可读性更强的XML格式。各个编程语言都有FTP client可以使用。  
创建此类型的Action Tracker时点击`Email FTP Username and Password`按钮， IR会把FTP服务器的用户名/密码发Advertiser的注册邮箱里。  
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
Items Conversion Request一般用于电商网站；Conversion Request则一般用于转运公司。当然，如果Advertiser想把一个运单也当作一件Item的话，用Items Conversion Request也无妨。  
数据文件上传成功后，Advertiser的注册邮箱里会收到一封IR发出的标题为`Data Submission Received`的邮件，里面是对此次上传的数据的处理结果。Batch Details中Failure为False时数据才算上传成功。
数据文件默认是上传到根目录下的，Advertiser无须考虑创建不同的文件夹来管理上传的数据文件，IR也没说允许这么做。  
更多信息请参考文档：[Batch FTP Tracking Method](http://support.impactradius.com/display/ADVERTISER/Batch+FTP+Tracking+Method)、[Advertiser FTP Conversion Reporting Method](http://support.impactradius.com/display/ADVERTISER/Guide+-+Tracking+Integration#Guide-TrackingIntegration-AdvertiserFTPConversionReportingMethodforOnlineActions)

- Web Services  
这里的Web Services不是通常意义上的Web Service，其实它就是个HTTP REST API。各个编程语言都有HTTP Client可以使用。  
请详细阅读文档[Web Services Tracking](http://support.impactradius.com/display/ADVERTISER/Web+Services+Tracking)中关于authenticate的内容。  
更多信息请参考文档：[Web Services Tracking](http://support.impactradius.com/display/ADVERTISER/Web+Services+Tracking)、[API-Conversions](http://dev.impactradius.com/display/api/Conversions)    
`纠错`：在文档[Web Services Tracking](http://support.impactradius.com/display/ADVERTISER/Web+Services+Tracking)中Conversions的api接口    
		
		https://api.impactradius.com/2010-09-01/Advertisers/{AccountSid}/Conversions
是错误的。正确的应该是	
  
 	  	https://api.impactradius.com/Advertisers/{AccountSid}/Conversions

###Action Tracker  
为各种不同的action定义相应的tracker。  
创建Action Tracker请到[这里](https://member.impactradius.com/secure/advertiser/campaign/actiontracker/view-actiontracker-flow.ihtml?execution=e1s1)。此页面会列出所有已经创建好的的Action Tracker，欲创建新的Action Tracker请点击`Create Action Tracker`按钮。  
前期在Advertiser不熟悉IR的情况下可由Extrabux协助创建Action Tracker，但我们希望后面Advertiser能够自己来掌控Action Tracker，毕竟这是IR里最经常用到的功能。  

下面我们重点说一下Action Tracker的测试。`在开始测试之前请Advertiser确保你们的开发人员已经拿到了IR系统的账号`。
######Action Tracker的测试  




[^1]:订单金额里可能包含优惠券、积分等内容，Advertiser可根据自身的业务需求来决定是否剔除它们。
[^2]:也可以是固定金额的佣金。
[^3]:这里虽然说的是入口页面，但其实只要有个后台就行了，不需要真的有对应的页面。因为它只要能接收click和irmpname参数，然后写到cookie里，然后直接302重定向到首页或其他页面就行了。而且强烈建议在这个后台做302重定向，不然一长串的参数在地址栏里确实不好看。它的职责就是接收IR传入的参数，参数处理完后就重定向到想去的页面。
[^4]:每次Ad click换会产生一个惟一的clickid，它是字母、数字和一些符号的组合。
[^5]:Impact Radius Media Partner Name
[^6]:IR内置的其他参数和Advertiser可自定义的参数请参考[Campaign Settings -> Online Traffic Settings -> Show advanced settings -> Query String Parameters](https://member.impactradius.com/secure/advertiser/campaign/technicalintegration-flow.ihtml?execution=e1s1)
[^7]:更多的可选项请参考[文档](http://support.impactradius.com/display/ADVERTISER/Batch+FTP+Tracking+Method#BatchFTPTrackingMethod-Fields)
[^8]:是的。Advertiser回传给IR的数据IR同样也会给到Extrabux，Extrabux再显示给用户看。真实有效的Order Id可以让用户很方便的和Advertiser里的订单/运单对应起来。
[^9]:时区缩写请参考[time_zone_abbreviations](https://en.wikipedia.org/wiki/List_of_time_zone_abbreviations)。如果你们系统使用的是中国标准时间，请不要使用CST来表示，因为CST可表示多个时区，而且在IR里也的确不用于表示Chinese Standard Time。请改用HKT(Hong Kong Time)。
[^10]:货币缩写请参考[currency-iso](http://www.currency-iso.org/dam/downloads/lists/list_one.xml)
[^11]:Batch FTP和Web Services这两种tracking method是延后回传数据。







