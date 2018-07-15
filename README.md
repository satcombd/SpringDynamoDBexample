## Technology

– Java 1.8
– Maven 3.3.9
– Spring Tool Suite – Version 3.9.0.RELEASE
– Spring Boot: 1.5.9.RELEASE
II. Overview
1. Goal

spring-data-dynamodb-goal
2. Project Structure

spring-data-dynamodb-structure

– Class Customer corresponds to entity and table Customer.
– CustomerRepository is an interface extends CrudRepository, will be autowired in WebController for implementing repository methods and custom finder methods.
– WebController is a REST Controller which has request mapping methods for RESTful requests such as: save, delete, findall, findbyid, findbylastname.
– Configuration for DynamoDB properties in application.properties. The accesskey and secretkey are just arbitrary values and are not needed to actually authenticate when accessing local instance of DynamoDB. The properties will be dynamically pulled out in the DynamoDBConfig.
– Dependencies for Spring Boot and DynamoDB in pom.xml
III. Practice
## Create Spring Boot project & add Dependencies

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
	<groupId>com.amazonaws</groupId>
	<artifactId>aws-java-sdk-dynamodb</artifactId>
	<version>1.11.34</version>
</dependency>

<dependency>
	<groupId>com.github.derjust</groupId>
	<artifactId>spring-data-dynamodb</artifactId>
	<version>4.5.0</version>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
 
<dependency>
	<groupId>com.amazonaws</groupId>
	<artifactId>aws-java-sdk-dynamodb</artifactId>
	<version>1.11.34</version>
</dependency>
 
<dependency>
	<groupId>com.github.derjust</groupId>
	<artifactId>spring-data-dynamodb</artifactId>
	<version>4.5.0</version>
</dependency>

## Configure Spring Data DynamoDB.

Open application.properties:
amazon.dynamodb.endpoint=http://localhost:8000/
amazon.aws.accesskey=jsa
amazon.aws.secretkey=javasampleapproach
	
amazon.dynamodb.endpoint=http://localhost:8000/
amazon.aws.accesskey=jsa
amazon.aws.secretkey=javasampleapproach

## Under package config, create DynamoDBConfig class:
package com.javasampleapproach.dynamodb.config;

import org.socialsignin.spring.data.dynamodb.repository.config.EnableDynamoDBRepositories;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient;
import com.amazonaws.util.StringUtils;

@Configuration
@EnableDynamoDBRepositories(basePackages = "com.javasampleapproach.dynamodb.repo")
public class DynamoDBConfig {

	@Value("${amazon.dynamodb.endpoint}")
	private String dBEndpoint;

	@Value("${amazon.aws.accesskey}")
	private String accessKey;

	@Value("${amazon.aws.secretkey}")
	private String secretKey;

	@Bean
	public AmazonDynamoDB amazonDynamoDB() {
		AmazonDynamoDB dynamoDB = new AmazonDynamoDBClient(amazonAWSCredentials());

		if (!StringUtils.isNullOrEmpty(dBEndpoint)) {
			dynamoDB.setEndpoint(dBEndpoint);
		}

		return dynamoDB;
	}

	@Bean
	public AWSCredentials amazonAWSCredentials() {
		return new BasicAWSCredentials(accessKey, secretKey);
	}
}
###
	
package com.javasampleapproach.dynamodb.config;
 
import org.socialsignin.spring.data.dynamodb.repository.config.EnableDynamoDBRepositories;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient;
import com.amazonaws.util.StringUtils;
 
@Configuration
@EnableDynamoDBRepositories(basePackages = "com.javasampleapproach.dynamodb.repo")
public class DynamoDBConfig {
 
	@Value("${amazon.dynamodb.endpoint}")
	private String dBEndpoint;
 
	@Value("${amazon.aws.accesskey}")
	private String accessKey;
 
	@Value("${amazon.aws.secretkey}")
	private String secretKey;
 
	@Bean
	public AmazonDynamoDB amazonDynamoDB() {
		AmazonDynamoDB dynamoDB = new AmazonDynamoDBClient(amazonAWSCredentials());
 
		if (!StringUtils.isNullOrEmpty(dBEndpoint)) {
			dynamoDB.setEndpoint(dBEndpoint);
		}
 
		return dynamoDB;
	}
 
	@Bean
	public AWSCredentials amazonAWSCredentials() {
		return new BasicAWSCredentials(accessKey, secretKey);
	}
}

3. Create DataModel Class

Under package model, create Customer class:
package com.javasampleapproach.dynamodb.model;

import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable;

@DynamoDBTable(tableName = "Customer")
public class Customer {

	private String id;
	private String firstName;
	private String lastName;

	public Customer() {
	}

	public Customer(String id, String firstName, String lastName) {
		this.id = id;
		this.firstName = firstName;
		this.lastName = lastName;
	}

	@DynamoDBHashKey(attributeName = "Id")
	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	@DynamoDBAttribute(attributeName = "FirstName")
	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	@DynamoDBAttribute(attributeName = "LastName")
	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	@Override
	public String toString() {
		return String.format("Customer[id=%s, firstName='%s', lastName='%s']", id, firstName, lastName);
	}
}

	
package com.javasampleapproach.dynamodb.model;
 
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable;
 
@DynamoDBTable(tableName = "Customer")
public class Customer {
 
	private String id;
	private String firstName;
	private String lastName;
 
	public Customer() {
	}
 
	public Customer(String id, String firstName, String lastName) {
		this.id = id;
		this.firstName = firstName;
		this.lastName = lastName;
	}
 
	@DynamoDBHashKey(attributeName = "Id")
	public String getId() {
		return id;
	}
 
	public void setId(String id) {
		this.id = id;
	}
 
	@DynamoDBAttribute(attributeName = "FirstName")
	public String getFirstName() {
		return firstName;
	}
 
	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}
 
	@DynamoDBAttribute(attributeName = "LastName")
	public String getLastName() {
		return lastName;
	}
 
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
 
	@Override
	public String toString() {
		return String.format("Customer[id=%s, firstName='%s', lastName='%s']", id, firstName, lastName);
	}
}

## 4. Create Repository Interface

This interface helps us do all CRUD functions:
package com.javasampleapproach.dynamodb.repo;

import java.util.List;

import org.socialsignin.spring.data.dynamodb.repository.EnableScan;
import org.springframework.data.repository.CrudRepository;

import com.javasampleapproach.dynamodb.model.Customer;

@EnableScan
public interface CustomerRepository extends CrudRepository<Customer, String> {

	List<Customer> findByLastName(String lastName);
}


package com.javasampleapproach.dynamodb.repo;
 
import java.util.List;
 
import org.socialsignin.spring.data.dynamodb.repository.EnableScan;
import org.springframework.data.repository.CrudRepository;
 
import com.javasampleapproach.dynamodb.model.Customer;
 
@EnableScan
public interface CustomerRepository extends CrudRepository<Customer, String> {
 
	List<Customer> findByLastName(String lastName);
}

5. Create Web Controller

The controller receives requests from client, using repository to add/get data and return results:
package com.javasampleapproach.dynamodb.controller;

import java.util.Arrays;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.javasampleapproach.dynamodb.model.Customer;
import com.javasampleapproach.dynamodb.repo.CustomerRepository;

@RestController
public class WebController {

	@Autowired
	CustomerRepository repository;

	@RequestMapping("/delete")
	public String delete() {
		repository.deleteAll();
		return "Done";
	}

	@RequestMapping("/save")
	public String save() {
		// save a single Customer
		repository.save(new Customer("JSA-1", "Jack", "Smith"));

		// save a list of Customers
		repository.save(Arrays.asList(new Customer("JSA-2", "Adam", "Johnson"), new Customer("JSA-3", "Kim", "Smith"),
				new Customer("JSA-4", "David", "Williams"), new Customer("JSA-5", "Peter", "Davis")));

		return "Done";
	}

	@RequestMapping("/findall")
	public String findAll() {
		String result = "";
		Iterable<Customer> customers = repository.findAll();

		for (Customer cust : customers) {
			result += cust.toString() + "<br>";
		}

		return result;
	}

	@RequestMapping("/findbyid")
	public String findById(@RequestParam("id") String id) {
		String result = "";
		result = repository.findOne(id).toString();
		return result;
	}

	@RequestMapping("/findbylastname")
	public String fetchDataByLastName(@RequestParam("lastname") String lastName) {
		String result = "";

		for (Customer cust : repository.findByLastName(lastName)) {
			result += cust.toString() + "<br>";
		}

		return result;
	}
}

	
package com.javasampleapproach.dynamodb.controller;
 
import java.util.Arrays;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
 
import com.javasampleapproach.dynamodb.model.Customer;
import com.javasampleapproach.dynamodb.repo.CustomerRepository;
 
@RestController
public class WebController {
 
	@Autowired
	CustomerRepository repository;
 
	@RequestMapping("/delete")
	public String delete() {
		repository.deleteAll();
		return "Done";
	}
 
	@RequestMapping("/save")
	public String save() {
		// save a single Customer
		repository.save(new Customer("JSA-1", "Jack", "Smith"));
 
		// save a list of Customers
		repository.save(Arrays.asList(new Customer("JSA-2", "Adam", "Johnson"), new Customer("JSA-3", "Kim", "Smith"),
				new Customer("JSA-4", "David", "Williams"), new Customer("JSA-5", "Peter", "Davis")));
 
		return "Done";
	}
 
	@RequestMapping("/findall")
	public String findAll() {
		String result = "";
		Iterable<Customer> customers = repository.findAll();
 
		for (Customer cust : customers) {
			result += cust.toString() + "<br>";
		}
 
		return result;
	}
 
	@RequestMapping("/findbyid")
	public String findById(@RequestParam("id") String id) {
		String result = "";
		result = repository.findOne(id).toString();
		return result;
	}
 
	@RequestMapping("/findbylastname")
	public String fetchDataByLastName(@RequestParam("lastname") String lastName) {
		String result = "";
 
		for (Customer cust : repository.findByLastName(lastName)) {
			result += cust.toString() + "<br>";
		}
 
		return result;
	}
}

6. Create DynamoDB table

– Run DynamoDB:
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb


– Create Customer Table:
aws dynamodb create-table --table-name Customer --attribute-definitions AttributeName=Id,AttributeType=S --key-schema AttributeName=Id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 --endpoint-url http://localhost:8000

Response is like:
{
    "TableDescription": {
        "TableArn": "arn:aws:dynamodb:ddblocal:000000000000:table/Customer",
        "AttributeDefinitions": [
            {
                "AttributeName": "Id",
                "AttributeType": "S"
            }
        ],
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "WriteCapacityUnits": 1,
            "LastIncreaseDateTime": 0.0,
            "ReadCapacityUnits": 1,
            "LastDecreaseDateTime": 0.0
        },
        "TableSizeBytes": 0,
        "TableName": "Customer",
        "TableStatus": "ACTIVE",
        "KeySchema": [
            {
                "KeyType": "HASH",
                "AttributeName": "Id"
            }
        ],
        "ItemCount": 0,
        "CreationDateTime": 1514538274.168
    }
}

	
{
    "TableDescription": {
        "TableArn": "arn:aws:dynamodb:ddblocal:000000000000:table/Customer",
        "AttributeDefinitions": [
            {
                "AttributeName": "Id",
                "AttributeType": "S"
            }
        ],
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "WriteCapacityUnits": 1,
            "LastIncreaseDateTime": 0.0,
            "ReadCapacityUnits": 1,
            "LastDecreaseDateTime": 0.0
        },
        "TableSizeBytes": 0,
        "TableName": "Customer",
        "TableStatus": "ACTIVE",
        "KeySchema": [
            {
                "KeyType": "HASH",
                "AttributeName": "Id"
            }
        ],
        "ItemCount": 0,
        "CreationDateTime": 1514538274.168
    }
}

## 7. Run Spring Boot Application & Check Result

– Config maven build:
clean install
– Run project with mode Spring Boot App
– Check results:

Request 1
http://localhost:8080/save
The browser returns Done and if checking database with Customer table, we can see some data rows has been added:
spring-data-dynamodb-table-result-save

Request 2
http://localhost:8080/findall
spring-data-dynamodb-table-result-findall

Request 3
http://localhost:8080/findbyid?id=JSA-3
spring-data-dynamodb-table-result-findbyid

Request 4
http://localhost:8080/findbylastname?lastname=Smith
spring-data-dynamodb-table-result-findbylastname

Request 5
http://localhost:8080/delete
The browser returns Done and if checking database with Customer table, we can see:
spring-data-dynamodb-table-result-delete
8. Way to check DynamoDB Table

Go to http://localhost:8000/shell, then run the code below:
var dynamodb = new AWS.DynamoDB({
region: 'us-east-1',
endpoint: "http://localhost:8000"
});
var tableName = "Customer";

var params = {
TableName: tableName,
Select: "ALL_ATTRIBUTES"
};

function doScan(response) {
	if (response.error) ppJson(response.error); // an error occurred
	else {
		ppJson(response.data); // successful response

		// More data.  Keep calling scan.
		if ('LastEvaluatedKey' in response.data) {
			response.request.params.ExclusiveStartKey = response.data.LastEvaluatedKey;
			dynamodb.scan(response.request.params)
				.on('complete', doScan)
				.send();
		}
	}
}

console.log("Starting a Scan of the table");
dynamodb.scan(params)
.on('complete', doScan)
.send();

function doScan(response) {
	if (response.error) ppJson(response.error); // an error occurred
	else {
		ppJson(response.data); // successful response
 
		// More data.  Keep calling scan.
		if ('LastEvaluatedKey' in response.data) {
			response.request.params.ExclusiveStartKey = response.data.LastEvaluatedKey;
			dynamodb.scan(response.request.params)
				.on('complete', doScan)
				.send();
		}
	}
}
 
console.log("Starting a Scan of the table");
dynamodb.scan(params)
.on('complete', doScan)
.send();
