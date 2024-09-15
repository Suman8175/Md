## Step 1: Add dependency `webSocket`

## Step 2: Add configuration file
   - Implement `WebSocketMessageBrokerConfigurer `class
     ```java
        @Configuration
        @EnableWebSocketMessageBroker
        public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
            @Override
            public void registerStompEndpoints(StompEndpointRegistry registry) {
                registry.addEndpoint("/chat").withSockJS();
            }

            @Override
            public void configureMessageBroker(MessageBrokerRegistry registry) {
            registry.setApplicationDestinationPrefixes("/app");
            registry.enableSimpleBroker("/topic");
            }
        }
    ```