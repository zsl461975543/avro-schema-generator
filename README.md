[![Build Status](https://travis-ci.org/artur-tamazian/avro-schema-generator.svg?branch=master)](https://travis-ci.org/artur-tamazian/avro-schema-generator)
[![Coverage](https://codecov.io/gh/artur-tamazian/avro-schema-generator/branch/master/graph/badge.svg)](https://codecov.io/gh/artur-tamazian/avro-schema-generator)
[![Join the chat at https://gitter.im/avro-schema-generator](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/avro-schema-generator?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

# avro-schema-generator
Library for generating avro schema files (.avsc) based on DB tables structure.

## How it works:

1. Prepare a db connection URL, username and a password. Use these to create a DbSchemaExtractor
2. If you're fancy, configure FormatterConfig and AvroConfig.
3. Tell schema extractor to give you avro models for tables that you're interested in.
4. Pass those to SchemaGenerator which will give you perfect avro schemas.

## Here's basic example:

```java
// Initialize db crawler that will create avro model objects
DbSchemaExtractor schemaExtractor = new DbSchemaExtractor("jdbc:mysql://localhost:3306", "root", "pass");

AvroConfig avroConfig = new AvroConfig("some.namespace");
// Get avro models for a few tables
List<AvroSchema> schemas = schemaExtractor.getForTables(avroConfig, "mydb", "users", "payments");

for (AvroSchema schema : schemas) {
    // Pass avro model to SchemaGenerator, get schema and print it out.
    String schemaJson = SchemaGenerator.generate(schema);
    System.out.println(schemaJson);
}
```

## More complex example:

```java
DbSchemaExtractor schemaExtractor = new DbSchemaExtractor("jdbc:mysql://localhost:3306", "root", "pass");

// Some of available configuration options
AvroConfig avroConfig = new AvroConfig("some.namespace")
    .setRepresentEnumsAsStrings(true) // use 'string' avro type instead of 'enum' for enums
    .setAllFieldsDefaultNull(true)    // adds default: 'null' to fields definition
    .setNullableTrueByDefault(true)   // makes all fields nullable
    .setSchemaNameMapper(new ToCamelCase().andThen(new RemovePlural())) // specify table name transformation to be used for schema name
    .setUnknownTypeResolver(type -> "string") // specify what to do with custom and unsupported db types
    .setDateTypeClass(Date.class) // add hint for avro compiler about which class to use for dates
    .setAvroSchemaPostProcessor((schema, table) -> {
        // adding some custom properties to avro schema
        schema.addCustomProperty("db-schema-name", "mydb");
        schema.addCustomProperty("db-table-name", table.getName());
});

// Get avro models for a few tables
List<AvroSchema> schemas = schemaExtractor.getAll(avroConfig);

// You can specify some formatting options by creating a FormatterConfig and passing it to SchemaGenerator.
FormatterConfig formatterConfig = FormatterConfig.builder()
            .setPrettyPrintSchema(true)
            .setPrettyPrintFields(false)
            .setIndent("    ")
            .build();

for (AvroSchema schema : schemas) {
    String schemaJson = SchemaGenerator.generate(schema, formatterConfig);
    System.out.println(schemaJson);
}
```

## Maven dependency

```
<dependency>
  <groupId>com.github.artur-tamazian</groupId>
  <artifactId>avro-schema-generator</artifactId>
  <version>1.0.2</version>
</dependency>
```

## Supported databases

DB crawling is done using the SchemaCrawler http://www.schemacrawler.com/. So it should work fine with RDBMS mentioned here: http://www.schemacrawler.com/database-support.html.

avro-schema-generator itself was successfully used with MySQL and PostgreSQL.

## TODO / Suggested contributions

- Migrate to newer version of [Schema Crawler](https://github.com/schemacrawler/SchemaCrawler)
- Add basic toString() implementations to models classes.
  - Consider using [Project Lombok](https://projectlombok.org/features/Data)
