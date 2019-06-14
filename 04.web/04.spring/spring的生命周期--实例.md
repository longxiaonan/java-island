在spring启动的时候初始化服务
```java
package com.vdp.vlr.init;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

import com.vdp.common.utils.TimeScheduler;
import com.vdp.core.component.rabbitmq.config.RabbitmqProps;
import com.vdp.vlr.cache.DataUpdateHandler;
import com.vdp.vlr.cache.RealTimeDataMgr;
import com.vdp.vlr.websocket.WebSocketServer;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
//@ConditionalOnBean(SpringContextUtils.class)
public class VLRInitApplicationListener implements ApplicationListener<ApplicationReadyEvent>{
	
	RabbitmqProps rabbitmqProps;

	@Autowired
	public VLRInitApplicationListener(RabbitmqProps rabbitmqProps) {
		this.rabbitmqProps = rabbitmqProps;
	}
	
	@Override
//	public void onApplicationEvent(ContextRefreshedEvent event) {
	public void onApplicationEvent(ApplicationReadyEvent contextRefreshedEvent) {
		try {
			init();
		} catch (Exception e) {
			log.error("启动服务异常");
			log.error(e.getMessage(),e);
		}
	}
	
	public void init() throws Exception {
		
		log.info("################初始化启动定时器组件TimeScheduler################");
		TimeScheduler.instance().start(2);

		/**
		 * 以下是启动一个Sokect服务，监听和订阅“SCDATA”的数据， 使用SubscribeDataEventHandler处理sokect
		 * 事件，将数据PUT到“数据处理服务”队列 (DataHandleSvc)
		 * 
		 **/
		// 启动数据管理服务
		RealTimeDataMgr.getInstance();

		//application.yml配置文件中队列的key,如队列配置为dataUpdateQ: topic.scdata.sc.vlr,那么queueKey为dataUpdateQ
		DataUpdateHandler.getInst().start("dataUpdateQ");

		// 启动查询服务
		/*QueryRpcSvc QueryRpcSvc;
		QueryRpcSvc = new QueryRpcSvc();

		try {
			String queryUrl = properties.getProperties("Server_VLR_QUERY_URL", "mina#127.0.0.1:9702:-1");
			QueryRpcSvc.start(queryUrl);
			submitServerAttribute(SvcTypeConst.VLR_QUERY.getSvcName(), queryUrl);
		} catch (Exception e) {
			logger.logError(e);
		}*/

		// 启动WebSocket服务端
		// new WebSocketServer(9992, "/websocketVlr").startServer();
//		new WebSocketServer().loadProperties().startServer();
		WebSocketServer.getInstance().startServer();
		
		log.info("开启服务");
	}

}
```
