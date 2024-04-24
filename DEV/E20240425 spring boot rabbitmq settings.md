## E20240425 spring boot rabbitmq settings.md

picup 유물 

``` java
package ecsimsw.picup.mq;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
public class MessageRouteConfig {

    public static final String GLOBAL_EXCHANGE_NAME = "global.exchange";
    public static final String DEAD_LETTER_EXCHANGE_NAME = "dead.letter.exchange";

    public static final String FILE_DELETE_ALL_QUEUE_NAME = "file.deleteAll.queue";
    public static final String FILE_DELETE_ALL_QUEUE_KEY = "file.deleteAll";

    public static final String FILE_DELETION_RECOVER_QUEUE_NAME = "file.deletion.recover.queue";
    public static final String FILE_DELETION_RECOVER_QUEUE_KEY = "file.deletion.recover";

    @Bean
    public DirectExchange globalExchange() {
        return new DirectExchange(GLOBAL_EXCHANGE_NAME);
    }

    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange(DEAD_LETTER_EXCHANGE_NAME);
    }

    @Bean
    public Queue fileDeleteAllQueue() {
        return QueueBuilder.durable(FILE_DELETE_ALL_QUEUE_NAME)
            .deadLetterExchange(DEAD_LETTER_EXCHANGE_NAME)
            .withArguments(Map.of(
                "x-dead-letter-exchange", DEAD_LETTER_EXCHANGE_NAME,
                "x-dead-letter-routing-key", FILE_DELETION_RECOVER_QUEUE_KEY
            ))
            .build();
    }

    @Bean
    public Queue fileDeletionRecoverQueue() {
        return QueueBuilder.durable(FILE_DELETION_RECOVER_QUEUE_NAME).build();
    }

    @Bean
    public Binding fileDeleteAllQueueBinding(
        DirectExchange globalExchange,
        Queue fileDeleteAllQueue
    ) {
        return BindingBuilder
            .bind(fileDeleteAllQueue)
            .to(globalExchange)
            .with(FILE_DELETE_ALL_QUEUE_KEY);
    }

    @Bean
    public Binding fileDeletionRecoverQueueBinding(
        DirectExchange deadLetterExchange,
        Queue fileDeletionRecoverQueue
    ) {
        return BindingBuilder
            .bind(fileDeletionRecoverQueue)
            .to(deadLetterExchange)
            .with(FILE_DELETION_RECOVER_QUEUE_KEY);
    }
}
```

``` java
package ecsimsw.picup.mq;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.Connection;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    private static final Logger LOGGER = LoggerFactory.getLogger(RabbitMQConfig.class);

    public static final int CONNECTION_RETRY_COUNT = 2;

    @Bean
    public RabbitTemplate rabbitTemplate(
        ConnectionFactory connectionFactory
    ) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(messageConverter());
        return rabbitTemplate;
    }

    @Bean
    public ConnectionFactory connectionFactory(
        @Value("${spring.rabbitmq.host}") String host,
        @Value("${spring.rabbitmq.port}") int port,
        @Value("${spring.rabbitmq.username}") String username,
        @Value("${spring.rabbitmq.password}") String password
    ) {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();   //  return same connection from all createConnection() calls, and ignores calls to Connection.close() and caches Channel
        connectionFactory.setHost(host);
        connectionFactory.setPort(port);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.addConnectionListener(new ConnectionListener() {
            @Override
            public void onCreate(Connection connection) {
                LOGGER.info("Message queue server connection is created");
            }

            @Override
            public void onClose(Connection connection) {
                LOGGER.error("Message queue server connection is created");
            }
        });
        return connectionFactory;
    }

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

``` java
package ecsimsw.picup.mq;

import org.springframework.amqp.rabbit.config.RetryInterceptorBuilder;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.RabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.retry.RejectAndDontRequeueRecoverer;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;

@Configuration
public class DeletionQueueContainerFactories {

    public static final String FILE_DELETION_QUEUE_CF = "fileDeletionQueueContainerFactory";

    private static final int CONCURRENT_CONSUMERS = 5;
    private static final int RETRY_MAX_ATTEMPTS = 3;
    private static final int RETRY_INITIAL_INTERVAL_SEC = 1;
    private static final int RETRY_MULTIPLIER = 3;
    private static final int RETRY_MAX_INTERVAL_SEC = 10;
    private static final int PREFETCH_COUNT = 5;

    @Bean
    public RabbitListenerContainerFactory<SimpleMessageListenerContainer> fileDeletionQueueContainerFactory(
        ConnectionFactory connectionFactory,
        MessageConverter messageConverter
    ) {
        var factory = new SimpleRabbitListenerContainerFactory();
        factory.setMessageConverter(messageConverter);
        factory.setPrefetchCount(PREFETCH_COUNT);
        factory.setConnectionFactory(connectionFactory);
        factory.setConcurrentConsumers(CONCURRENT_CONSUMERS);
        factory.setAdviceChain(RetryInterceptorBuilder.stateless()
            .maxAttempts(RETRY_MAX_ATTEMPTS)
            .backOffOptions(
                Duration.ofSeconds(RETRY_INITIAL_INTERVAL_SEC).toMillis(),
                RETRY_MULTIPLIER,
                Duration.ofSeconds(RETRY_MAX_INTERVAL_SEC).toMillis()
            )
            .recoverer(new RejectAndDontRequeueRecoverer())
            .build());
        return factory;
    }
}
```

``` java
package ecsimsw.picup.album.controller;

import ecsimsw.picup.album.service.FileService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Controller;

import java.util.List;

import static ecsimsw.picup.mq.DeletionQueueContainerFactories.FILE_DELETION_QUEUE_CF;
import static ecsimsw.picup.mq.MessageRouteConfig.FILE_DELETE_ALL_QUEUE_NAME;
import static ecsimsw.picup.mq.MessageRouteConfig.FILE_DELETION_RECOVER_QUEUE_NAME;

@Controller
public class FileDeleteMessageQueueListener {

    private static final Logger LOGGER = LoggerFactory.getLogger(FileDeleteMessageQueueListener.class);

    private final FileService fileService;

    public FileDeleteMessageQueueListener(FileService fileService) {
        this.fileService = fileService;
    }

    @RabbitListener(queues = FILE_DELETE_ALL_QUEUE_NAME, containerFactory = FILE_DELETION_QUEUE_CF)
    public void deleteAll(List<String> resources) {
        LOGGER.info("Delete file : " + String.join("\n ", resources));
        resources.forEach(fileService::delete);
    }

    @RabbitListener(queues = FILE_DELETION_RECOVER_QUEUE_NAME)
    public void deleteAllRecover(Message failedMessage) {
        var alertMessage = "dead letter from file deletion queue \n" + "body : " + failedMessage.getPayload();
        LOGGER.error(alertMessage);
    }
}
```
