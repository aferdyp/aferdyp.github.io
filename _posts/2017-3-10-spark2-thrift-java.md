---
layout: post
comments: false
title:  "Accessing Spark 2.x Thrift server via jdbc"
date:   2017-03-10 00:00:00 +0000
categories: Spark Thrift java
draft: true
---

    <dependencies>
      <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.11</artifactId>
        <version>2.1.0</version>
      </dependency>

      <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.3</version>
      </dependency>

      <dependency>
        <groupId>org.spark-project.hive</groupId>
        <artifactId>hive-jdbc</artifactId>
        <version>1.2.1.spark2</version>
      </dependency>
    </dependencies>

    public class App {
        public static void main( String[] args ) {
            Connection conn;
            Statement stmt;
            String query;
            ResultSet rs;

            try {
                Class.forName("org.apache.hive.jdbc.HiveDriver").newInstance();
                String url = "jdbc:hive2://sparkmaster:10000";
                conn = DriverManager.getConnection(url, "user", "");
                stmt = conn.createStatement();

                query = "show tables";
                rs = stmt.executeQuery(query);
                while (rs.next()) {
                    System.out.println(rs.getString(2));
                }
                conn.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
