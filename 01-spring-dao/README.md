# Spring Dao
**Sample project to demonstrate spring dao**

A simple JAX-RS project built on top of Grizzly server to show the usages of Spring Dao functionality. 


## Getting started

- In the Spring-Datasource.xml, configure the connection details with required number of connections to be pooled

```xml
    <bean id="dataSource" class ="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
		<property name="url" value="jdbc:mysql://CONNECTION_URL"/>
		<property name="username" value="root"/>
		<property name="password" value="password"/>	
		<property name="initialSize" value="10" />        
        <property name="maxIdle" value="10" />
        <property name="minIdle" value="5" />
	</bean>
```

- Define the model class
```java
public class Category {
	private int id;
	private String name;
	private byte[] image;
	private Boolean root;
	private Category parent;
	private List<Category> children;
}
```
- Implement a RowMapper class for mapping the fields between model class and SQL table

```java
private final class CategoryMapper implements RowMapper<Category> {

		@Override
		public Category mapRow(ResultSet rs, int rowNum) throws SQLException {
			Category category = new Category();
			category.setId(rs.getInt("category_id"));
			category.setName(rs.getString("category_name"));
			Blob image = rs.getBlob("category_image");
			if (image != null) {
				byte[] imageBytes = image.getBytes(0, (int)image.length());			
				image.free();
				category.setImage(imageBytes);
			}
			category.setRoot(rs.getBoolean("category_root"));
			// Get all the child categories
			CategoryDao categoryDao = context.getBean(CategoryDao.class);
			List<Category> children = categoryDao.listChildren(category.getId());
			category.setChildren(children);
			return category;
		}
		
	}
```
- Implement the DAO using the NamedParameterJdbcTemplate
```java
@Override
	public Category read(int id) {
		String query = "SELECT * FROM vl_category WHERE category_id = :id";
		SqlParameterSource paramSource = new MapSqlParameterSource()
				.addValue("id", id);
		List<Category> categories = jdbcTemplate.query(query, paramSource, new CategoryMapper());
		Category selectedCategory = null;
		if (categories.size() == 1) {
			selectedCategory = categories.get(0);
		}		
		return selectedCategory;
	}
```

## Build

In order build your own jar, run below maven command

```
    mvn clean install
```

## Running the server

```
    mvn exec:java
```

Open http://localhost:8080/docs/ in a browser and test the resource using swagger-ui.

## Authors

* **Venkatesh Mohanram** - *Initial work* - [venkatesh-mohanram](https://github.com/venkatesh-mohanram)
