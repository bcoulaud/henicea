# Henicea

Henicea was the brother of Cassandra in the Greek Mythology.

## Migration

Henicea provides a simple migration mechanism for Java projects.

Example config class for Spring Boot

```java
@Configuration
@Slf4j
public class CassandraConfig {

    @Autowired
    private ResourcePatternResolver resourceResolver;

    @Autowired
    private Migrator migrator;

    @Bean
    public Cluster cluster() throws IOException {
        LetterConfig.CassandraConfig cassandraConfig = config.getCassandra();

        log.info("Connecting to Cassandra using config {}", cassandraConfig);

        Cluster cluster = Cluster.builder()
                .addContactPoints(cassandraConfig.getContactpoints().split(","))
                .withPort(cassandraConfig.getPort())
                .build();

        log.info("Running migrations");
        migrator.execute(cluster, cassandraConfig.getKeyspace(), getMigrations());

        return cluster;
    }

    @Bean
    public Session session() throws Exception {
        return cluster().connect(config.getCassandra().getKeyspace());
    }

    private Resource[] getMigrations() throws IOException {
        return resourceResolver.getResources("classpath:/cassandra/*.cql");
    }
}
```

## Health check

Hernicea provides a simple health check through Spring Boot Actuator. The only requirement
is to have a Cassandra `Session` object in the Spring context.

To auto configure a health check add
```java
    @Bean
    public CassandraHealthIndicator cassandraHealthIndicator() {
        return new CassandraHealthIndicator();
    }
```
to your Java config class.