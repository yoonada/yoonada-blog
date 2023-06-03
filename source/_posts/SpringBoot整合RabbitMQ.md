---
title: SpringBoot整合RabbitMQ
date: 2022-09-26 17:33:48
tags: [SpringBoot,RabbitMQ]
categories: [SpringBoot,RabbitMQ]
---
#### 1、引入依赖
```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
```
#### 2、application.yml配置如下：
```yaml
spring:
  rabbitmq:
    host: xxx.xxx.xxx.xxx
    port: 5672
    password: guest
    username: guest
    virtual-host: /yoona-cloud
```
#### 3、枚举关系
```java
import com.yoona.cloud.common.core.base.IBaseEnum;
import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description: 队列枚举
 * @date 2022-06-17 11:25
 */
@Getter
@AllArgsConstructor
public enum RabbitMqQueueEnum implements IBaseEnum<String> {

    /**
     * 枚举所有队列
     */
    
    Q_MAIL_SEND("Q_MAIL_SEND", "邮件发送队列"),
    ;
    private final String value;

    private final String description;
}
```
```java
import com.yoona.cloud.common.core.base.IBaseEnum;
import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description: 交换机枚举
 * @date 2022-06-17 11:35
 */
@Getter
@AllArgsConstructor
public enum RabbitMqExchangeEnum implements IBaseEnum<String> {

    /**
     * 枚举所有交换机
     */

    E_TOPIC_MAIL_SEND("topic","topic类型的邮件发送交换机"),
    ;
    private final String value;

    private final String description;
}
```
```java
import com.yoona.cloud.common.core.base.IBaseEnum;
import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description: 路由枚举
 * @date 2022-06-17 11:39
 */
@Getter
@AllArgsConstructor
public enum RabbitMqRoutingKeyEnum implements IBaseEnum<String> {

    /**
     * 枚举所有路由
     */

    K_MAIL_SEND("K_MAIL_SEND","邮件发送路由键"),

    ;
    private final String value;

    private final String description;
}

```
```java
import com.yoona.cloud.common.core.base.IBaseEnum;
import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description: 绑定关系枚举
 * @date 2022-06-17 11:41
 */
@Getter
@AllArgsConstructor
public enum RabbitMqBindEnum implements IBaseEnum<String> {

    /**
     * 枚举所有绑定关系
     */

    MAIL_SEND(RabbitMqExchangeEnum.E_TOPIC_MAIL_SEND, RabbitMqQueueEnum.Q_MAIL_SEND, RabbitMqRoutingKeyEnum.K_MAIL_SEND,true ,"MAIL_SEND", "邮件发送"),

    ;

    private final RabbitMqExchangeEnum rabbitMqExchangeEnum;

    private final RabbitMqQueueEnum rabbitMqQueueEnum;

    private final RabbitMqRoutingKeyEnum rabbitMqRoutingKeyEnum;

    private final Boolean isBind;

    private final String value;

    private final String description;

}
```
```java

import lombok.Getter;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Exchange;
import org.springframework.amqp.core.TopicExchange;

import java.util.Arrays;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description:
 * @date 2022-06-17 11:54
 */
@Getter
public enum RabbitMqExchangeTypeEnum {

    /**
     * 根据交换机的类型，创建对应的交换机
     */

    DIRECT("direct") {
        @Override
        public Exchange createExchange(String exchangeName) {
            return new DirectExchange(exchangeName, true, false);
        }
    },

    TOPIC("topic") {
        @Override
        public Exchange createExchange(String exchangeName) {
            return new TopicExchange(exchangeName, true, false);
        }
    };

    public static RabbitMqExchangeTypeEnum getInstanceByType(String type) throws Exception {
        return Arrays.stream(RabbitMqExchangeTypeEnum.values()).filter(e -> e.getType().equals(type))
                .findAny()
                .orElseThrow(() -> new Exception("无效的exchange type"));
    }

    private final String type;


    RabbitMqExchangeTypeEnum(String type) {
        this.type = type;
    }

    /**
     * 创建交换机
     * @param exchangeName
     * @return
     */
    public abstract Exchange createExchange(String exchangeName);

}
```
#### 4、配置
```java

import com.yoona.cloud.common.rabbitmq.enums.RabbitMqBindEnum;
import com.yoona.cloud.common.rabbitmq.enums.RabbitMqExchangeEnum;
import com.yoona.cloud.message.mq.rabbit.enums.RabbitMqExchangeTypeEnum;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.Exchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.amqp.rabbit.transaction.RabbitTransactionManager;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import java.util.Arrays;

/**
 * @author YoonaDa
 * @email lintiaoda@suntang.com
 * @description:
 * @date 2022-06-17 11:50
 */
@Slf4j
@Configuration
@ConditionalOnClass(EnableRabbit.class)
public class RabbitMqConfiguration {

    @Resource

    private RabbitAdmin rabbitAdmin;

    public static final int DEFAULT_CONCURRENT = 10;

    @Bean("customContainerFactory")
    public SimpleRabbitListenerContainerFactory containerFactory(SimpleRabbitListenerContainerFactoryConfigurer configurer,
                                                                 ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConcurrentConsumers(DEFAULT_CONCURRENT);
        factory.setMaxConcurrentConsumers(DEFAULT_CONCURRENT);
        configurer.configure(factory, connectionFactory);
        return factory;
    }

    @Bean
    @ConditionalOnMissingBean
    public RabbitTransactionManager rabbitTransactionManager(ConnectionFactory connectionFactory) {
        return new RabbitTransactionManager(connectionFactory);
    }

    @Bean
    @ConditionalOnMissingBean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        return new RabbitAdmin(connectionFactory);
    }

    /**
     * 初始化
     */
    @PostConstruct
    protected void init() {
        // 创建交换机
        Arrays.stream(RabbitMqExchangeEnum.values())
                .forEach(rabbitMqExchangeEnum -> {
                    try {
                        Exchange exchange = RabbitMqExchangeTypeEnum
                                .getInstanceByType(rabbitMqExchangeEnum.getValue())
                                .createExchange(rabbitMqExchangeEnum.name());
                        rabbitAdmin.declareExchange(exchange);
                    } catch (Exception e) {
                        log.error("创建交换机时发生异常:{}", e.getMessage(), e);
                    }
                });
        // 创建队列并绑定exchange
        Arrays.stream(RabbitMqBindEnum.values()).forEach(rabbitMqBindEnum -> {
            if (!rabbitMqBindEnum.getIsBind()) {
                // 无需绑定
                return;
            }
            rabbitAdmin.declareQueue(new Queue(rabbitMqBindEnum.getRabbitMqQueueEnum().name()));
            rabbitAdmin.declareBinding(new Binding(
                    rabbitMqBindEnum.getRabbitMqQueueEnum().name(),
                    Binding.DestinationType.QUEUE,
                    rabbitMqBindEnum.getRabbitMqExchangeEnum().name(),
                    rabbitMqBindEnum.getRabbitMqRoutingKeyEnum().name(),
                    null));
        });

    }
}
```
#### 5、实际应用例子
```java
@Autowired
private RabbitTemplate rabbitTemplate;

rabbitTemplate.convertAndSend(
        RabbitMqExchangeEnum.E_TOPIC_MAIL_SEND.name(),
        RabbitMqRoutingKeyEnum.K_MAIL_SEND.name(),
        JSON.toJSONString(mailTask));
```
```java
@Slf4j
@Component
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
@RabbitListener(queues = {"Q_MAIL_SEND"})
public class MailSendListener{

    @RabbitHandler
    public void receive(String msg) {
        // 处理监到消息逻辑xxx...
    }
}
```