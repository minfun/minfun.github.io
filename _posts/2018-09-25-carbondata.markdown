spark-shell --jars /Users/leifanwang/Downloads/apache-carbondata-1.5.0-SNAPSHOT-bin-spark2.3.1-hadoop2.7.2.jar
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.CarbonSession._
val carbon = SparkSession.builder().config(sc.getConf).getOrCreateCarbonSession()
carbon.sql("CREATE TABLE IF NOT EXISTS usage_retention2(product_id LONG,country_code STRING,device_code STRING,market_code STRING,dim_age_id INTEGER,dim_gender_id INTEGER,date TIMESTAMP,est_retention_d0 DOUBLE,est_retention_d1 DOUBLE,est_retention_d2 DOUBLE,est_retention_d3 DOUBLE,est_retention_d4 DOUBLE,est_retention_d5 DOUBLE,est_retention_d6 DOUBLE,est_retention_d7 DOUBLE,est_retention_d14 DOUBLE,est_retention_d30 DOUBLE) STORED BY 'carbondata' TBLPROPERTIES('SORT_COLUMNS'='device_code, date, country_code, product_id','NO_INVERTED_INDEX'='est_retention_d0, est_retention_d1, est_retention_d2, est_retention_d3, est_retention_d4, est_retention_d5, est_retention_d6, est_retention_d7, est_retention_d14, est_retention_d30','DICTIONARY_INCLUDE' = 'market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id','SORT_SCOPE'='GLOBAL_SORT','CACHE_LEVEL'='BLOCKLET','TABLE_BLOCKSIZE'='256','GLOBAL_SORT_PARTITIONS'='2')")
carbon.sql("CREATE DATAMAP IF NOT EXISTS agg_by_month ON TABLE usage_retention USING 'timeSeries' DMPROPERTIES ('EVENT_TIME'='date','MONTH_GRANULARITY'='1') AS SELECT market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id, COUNT(date), COUNT(est_retention_d0), COUNT(est_retention_d1), COUNT(est_retention_d2), COUNT(est_retention_d3), COUNT(est_retention_d4), COUNT(est_retention_d5), COUNT(est_retention_d6), COUNT(est_retention_d7), COUNT(est_retention_d14), COUNT(est_retention_d30), AVG(est_retention_d0), MIN(est_retention_d0), MAX(est_retention_d0), AVG(est_retention_d1), MIN(est_retention_d1), MAX(est_retention_d1), AVG(est_retention_d2), MIN(est_retention_d2), MAX(est_retention_d2), AVG(est_retention_d3), MIN(est_retention_d3), MAX(est_retention_d3), AVG(est_retention_d4), MIN(est_retention_d4), MAX(est_retention_d4), AVG(est_retention_d5), MIN(est_retention_d5), MAX(est_retention_d5), AVG(est_retention_d6), MIN(est_retention_d6), MAX(est_retention_d6), AVG(est_retention_d7), MIN(est_retention_d7), MAX(est_retention_d7), AVG(est_retention_d14), MIN(est_retention_d14), MAX(est_retention_d14), AVG(est_retention_d30), MIN(est_retention_d30), MAX(est_retention_d30) FROM usage_retention GROUP BY market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id")
carbon.sql("CREATE DATAMAP IF NOT EXISTS agg_by_year ON TABLE usage_retention USING 'timeSeries' DMPROPERTIES ('EVENT_TIME'='date', 'YEAR_GRANULARITY'='1') AS SELECT market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id, COUNT(date), COUNT(est_retention_d0), COUNT(est_retention_d1), COUNT(est_retention_d2), COUNT(est_retention_d3), COUNT(est_retention_d4), COUNT(est_retention_d5), COUNT(est_retention_d6), COUNT(est_retention_d7), COUNT(est_retention_d14), COUNT(est_retention_d30), AVG(est_retention_d0), MIN(est_retention_d0), MAX(est_retention_d0), AVG(est_retention_d1), MIN(est_retention_d1), MAX(est_retention_d1), AVG(est_retention_d2), MIN(est_retention_d2), MAX(est_retention_d2), AVG(est_retention_d3), MIN(est_retention_d3), MAX(est_retention_d3), AVG(est_retention_d4), MIN(est_retention_d4), MAX(est_retention_d4), AVG(est_retention_d5), MIN(est_retention_d5), MAX(est_retention_d5), AVG(est_retention_d6), MIN(est_retention_d6), MAX(est_retention_d6), AVG(est_retention_d7), MIN(est_retention_d7), MAX(est_retention_d7), AVG(est_retention_d14), MIN(est_retention_d14), MAX(est_retention_d14), AVG(est_retention_d30), MIN(est_retention_d30), MAX(est_retention_d30) FROM usage_retention GROUP BY market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id")
carbon.sql("CREATE DATAMAP IF NOT EXISTS bloomfilter_all_dimensions ON TABLE usage_retention USING 'bloomfilter' DMPROPERTIES ('INDEX_COLUMNS'='market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id', 'BLOOM_SIZE'='640000', 'BLOOM_FPP'='0.000001', 'BLOOM_COMPRESS'='true')")
carbon.sql("describe formatted usage_retention").show(truncate=false)
carbon.sql("SHOW DATAMAP ON table usage_retention").show(truncate=false)



df=spark.read.parquet("s3a://b2b-prod-int-data-pipeline-unified/unified/app-ss.usageint.retention.v1/metric")
data_url='file:///services/appannie/aa-int-data-pipeline/aaintdatapipeline/tests/unit/application/app_ss/usageint_retention_v1/usage_retention.csv*'
df=spark.read.format('csv').option("header", "true").option('delimiter', '|').option("ignoreLeadingWhiteSpace", True).load(data_url)
not_null_df=df.filter(df.est_retention_d30.isNotNull())
not_null_df_count=not_null_df.count()
null_df=df.filter(df.est_retention_d30.isNull())
null_df_count=null_df.count()

data_url='file:///services/appannie/aa-int-data-pipeline/aaintdatapipeline/tests/unit/application/app_ss/usageint_retention_v1/usage_retention.csv*'
df=spark.read.format('csv').option("header", "true").option('delimiter', '|').option("ignoreLeadingWhiteSpace", True).load(data_url)
not_null_df=df.filter(df.est_retention_d30!='null')
not_null_df_count=not_null_df.count()
null_df=df.filter(df.est_retention_d30=='null')
null_df_count=null_df.count()
print 'not_null_df_count {}'.format(not_null_df_count)
print 'null_df_count {}'.format(null_df_count)
new_df_count=null_df.join(not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'new_df {}'.format(new_df_count)


from pyspark.storagelevel import StorageLevel
df=spark.read.parquet("s3a://b2b-prod-int-data-pipeline-unified/unified/app-ss.usageint.retention.v1/metric").persist(StorageLevel.MEMORY_AND_DISK)
d30_not_null_df=df.filter(df.est_retention_d30.isNotNull())
d30_not_null_df_count=d30_not_null_df.count()
d30_null_df=df.filter(df.est_retention_d30.isNull())
d30_null_df_count=d30_null_df.count()
d30_new_df_count=d30_null_df.join(d30_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd30_not_null_df_count {}'.format(d30_not_null_df_count)
print 'd30_null_df_count {}'.format(d30_null_df_count)
print 'd30_new_df {}'.format(d30_new_df_count)

d14_not_null_df=df.filter(df.est_retention_d14.isNotNull())
d14_not_null_df_count=d14_not_null_df.count()
d14_null_df=df.filter(df.est_retention_d14.isNull())
d14_null_df_count=d14_null_df.count()
d14_new_df_count=d14_null_df.join(d14_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd14_not_null_df_count {}'.format(d14_not_null_df_count)
print 'd14_null_df_count {}'.format(d14_null_df_count)
print 'd14_new_df {}'.format(d14_new_df_count)

d7_not_null_df=df.filter(df.est_retention_d7.isNotNull())
d7_not_null_df_count=d7_not_null_df.count()
d7_null_df=df.filter(df.est_retention_d7.isNull())
d7_null_df_count=d7_null_df.count()
d7_new_df_count=d7_null_df.join(d7_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd7_not_null_df_count {}'.format(d7_not_null_df_count)
print 'd7_null_df_count {}'.format(d7_null_df_count)
print 'd7_new_df {}'.format(d7_new_df_count)

d6_not_null_df=df.filter(df.est_retention_d6.isNotNull())
d6_not_null_df_count=d6_not_null_df.count()
d6_null_df=df.filter(df.est_retention_d6.isNull())
d6_null_df_count=d6_null_df.count()
d6_new_df_count=d6_null_df.join(d6_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd6_not_null_df_count {}'.format(d6_not_null_df_count)
print 'd6_null_df_count {}'.format(d6_null_df_count)
print 'd6_new_df {}'.format(d6_new_df_count)

d5_not_null_df=df.filter(df.est_retention_d5.isNotNull())
d5_not_null_df_count=d5_not_null_df.count()
d5_null_df=df.filter(df.est_retention_d5.isNull())
d5_null_df_count=d5_null_df.count()
d5_new_df_count=d5_null_df.join(d5_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd5_not_null_df_count {}'.format(d5_not_null_df_count)
print 'd5_null_df_count {}'.format(d5_null_df_count)
print 'd5_new_df {}'.format(d5_new_df_count)

d4_not_null_df=df.filter(df.est_retention_d4.isNotNull())
d4_not_null_df_count=d4_not_null_df.count()
d4_null_df=df.filter(df.est_retention_d4.isNull())
d4_null_df_count=d4_null_df.count()
d4_new_df_count=d4_null_df.join(d4_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd4_not_null_df_count {}'.format(d4_not_null_df_count)
print 'd4_null_df_count {}'.format(d4_null_df_count)
print 'd4_new_df {}'.format(d4_new_df_count)

d3_not_null_df=df.filter(df.est_retention_d3.isNotNull())
d3_not_null_df_count=d3_not_null_df.count()
d3_null_df=df.filter(df.est_retention_d3.isNull())
d3_null_df_count=d3_null_df.count()
d3_new_df_count=d3_null_df.join(d3_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd3_not_null_df_count {}'.format(d3_not_null_df_count)
print 'd3_null_df_count {}'.format(d3_null_df_count)
print 'd3_new_df {}'.format(d3_new_df_count)

d2_not_null_df=df.filter(df.est_retention_d2.isNotNull())
d2_not_null_df_count=d2_not_null_df.count()
d2_null_df=df.filter(df.est_retention_d2.isNull())
d2_null_df_count=d2_null_df.count()
d2_new_df_count=d2_null_df.join(d2_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd2_not_null_df_count {}'.format(d2_not_null_df_count)
print 'd2_null_df_count {}'.format(d2_null_df_count)
print 'd2_new_df {}'.format(d2_new_df_count)

d1_not_null_df=df.filter(df.est_retention_d1.isNotNull())
d1_not_null_df_count=d1_not_null_df.count()
d1_null_df=df.filter(df.est_retention_d1.isNull())
d1_null_df_count=d1_null_df.count()
d1_new_df_count=d1_null_df.join(d1_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd1_not_null_df_count {}'.format(d1_not_null_df_count)
print 'd1_null_df_count {}'.format(d1_null_df_count)
print 'd1_new_df {}'.format(d1_new_df_count)

d0_not_null_df=df.filter(df.est_retention_d0.isNotNull())
d0_not_null_df_count=d0_not_null_df.count()
d0_null_df=df.filter(df.est_retention_d0.isNull())
d0_null_df_count=d0_null_df.count()
d0_new_df_count=d0_null_df.join(d0_not_null_df, on=['product_id','dim_age_id','dim_gender_id','country_code','device_code','date','market_code'], how='left_outer').count()
print 'd0_not_null_df_count {}'.format(d0_not_null_df_count)
print 'd0_null_df_count {}'.format(d0_null_df_count)
print 'd0_new_df {}'.format(d0_new_df_count)


CREATE TABLE IF NOT EXISTS product_detail(product_id LONG,name STRING,company STRING,market_code STRING,dim_age_id INTEGER,dim_gender_id INTEGER,date TIMESTAMP,est_retention_d0 DOUBLE,est_retention_d1 DOUBLE,est_retention_d2 DOUBLE,est_retention_d3 DOUBLE,est_retention_d4 DOUBLE,est_retention_d5 DOUBLE,est_retention_d6 DOUBLE,est_retention_d7 DOUBLE,est_retention_d14 DOUBLE,est_retention_d30 DOUBLE) STORED BY 'carbondata' TBLPROPERTIES('SORT_COLUMNS'='market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id','NO_INVERTED_INDEX'='est_retention_d0, est_retention_d1, est_retention_d2, est_retention_d3, est_retention_d4, est_retention_d5, est_retention_d6, est_retention_d7, est_retention_d14, est_retention_d30','DICTIONARY_INCLUDE' = 'market_code, device_code, dim_age_id, dim_gender_id, date, country_code, product_id','SORT_SCOPE'='GLOBAL_SORT','CACHE_LEVEL'='BLOCKLET','TABLE_BLOCKSIZE'='256','GLOBAL_SORT_PARTITIONS'='2')

carbon.sql("CREATE TABLE IF NOT EXISTS product_detail(product_id LONG,market_code STRING,name STRING,company STRING,release_date STRING,price DOUBLE,version STRING,description STRING,store_id INTEGER,company_id INTEGER,category_id INTEGER,ss_urls STRING,size DOUBLE,web_urls STRING,created STRING,content_rating STRING,privacy_policy_url STRING,last_updated STRING,has_iap BOOLEAN,status INTEGER,current_release_date STRING,original_price DOUBLE,sensitive_status INTEGER,artwork_url STRING,slug STRING,scrape_reviews BOOLEAN,date_scraped STRING,scrape_failed INTEGER,dead BOOLEAN,sku STRING,req_version STRING,req_device INTEGER,has_game_center BOOLEAN,is_mac BOOLEAN,languages STRING,support_url STRING,license_url STRING,link_apps STRING,scrape_review_delay INTEGER,requirements STRING,app_store_notes STRING,bundle_id STRING,product_type INTEGER,bundle_product_count INTEGER,family_sharing BOOLEAN,purchased_separately_price DOUBLE,seller STRING,required_devices STRING,has_imsg BOOLEAN,is_hidden_from_springboard BOOLEAN,subtitle STRING,promotional_text STRING,editorial_badge_type STRING,editorial_badge_name STRING,only_32_bit BOOLEAN,class STRING,installs STRING,require_os STRING,downloads_chart_url STRING,video_url STRING,icon_url STRING,banner_image_url STRING,permissions STRING,whats_new STRING,related_apps STRING,also_installed_apps STRING,more_from_developer_apps STRING,is_publisher_top BOOLEAN,publisher_email STRING,scrape_review_status INTEGER,company_code STRING,source STRING) STORED BY 'carbondata' TBLPROPERTIES('SORT_COLUMNS'='has_imsg, has_iap, only_32_bit, has_game_center, is_mac, status, product_type, market_code, category_id, store_id, languages, company_code, company_id, product_id, class, name, company','NO_INVERTED_INDEX'='req_version, req_device, price, size, version, content_rating, original_price, bundle_product_count, purchased_separately_price, installs, scrape_review_status, version, description, ss_urls, web_urls, privacy_policy_url, artwork_url, slug, support_url, license_url, requirements, app_store_notes, family_sharing, required_devices, subtitle, promotional_text, editorial_badge_type, editorial_badge_name, downloads_chart_url, video_url, icon_url, banner_image_url, whats_new, related_apps, also_installed_apps, more_from_developer_apps, publisher_email','DICTIONARY_INCLUDE'='require_os','SORT_SCOPE'='LOCAL_SORT','CACHE_LEVEL'='BLOCKLET','TABLE_BLOCKSIZE'='256','GLOBAL_SORT_PARTITIONS'='2')")


def product_ddl(carbon: SparkSession) = {
    carbon.sql("DROP TABLE IF EXISTS productv1").show()
    carbon.sql(
      s"""
         |CREATE TABLE IF NOT EXISTS productv1(
         |market_code STRING,
         |product_id LONG,
         |country_code STRING,
         |category_id LONG,
         |company_id LONG,
         |name STRING,
         |company STRING,
         |release_date STRING,
         |price DOUBLE,
         |version STRING,
         |description STRING,
         |ss_urls STRING,
         |size DOUBLE,
         |web_urls STRING,
         |created STRING,
         |content_rating STRING,
         |privacy_policy_url STRING,
         |last_updated STRING,
         |has_iap BOOLEAN,
         |status LONG,
         |current_release_date STRING,
         |original_price DOUBLE,
         |sensitive_status LONG,
         |artwork_url STRING,
         |slug STRING,
         |scrape_reviews BOOLEAN,
         |date_scraped STRING,
         |scrape_failed LONG,
         |dead BOOLEAN,
         |sku STRING,
         |req_version STRING,
         |req_device LONG,
         |has_game_center BOOLEAN,
         |is_mac BOOLEAN,
         |languages STRING,
         |support_url STRING,
         |license_url STRING,
         |link_apps STRING,
         |scrape_review_delay LONG,
         |requirements STRING,
         |app_store_notes STRING,
         |bundle_id STRING,
         |product_type LONG,
         |bundle_product_count LONG,
         |family_sharing BOOLEAN,
         |purchased_separately_price DOUBLE,
         |seller STRING,
         |required_devices STRING,
         |has_imsg BOOLEAN,
         |is_hidden_from_springboard BOOLEAN,
         |subtitle STRING,
         |promotional_text STRING,
         |editorial_badge_type STRING,
         |editorial_badge_name STRING,
         |only_32_bit BOOLEAN,
         |class STRING,
         |installs STRING,
         |require_os STRING,
         |downloads_chart_url STRING,
         |video_url STRING,
         |icon_url STRING,
         |banner_image_url STRING,
         |permissions STRING,
         |whats_new STRING,
         |related_apps STRING,
         |also_installed_apps STRING,
         |more_from_developer_apps STRING,
         |is_publisher_top BOOLEAN,
         |publisher_email STRING,
         |scrape_review_status LONG,
         |company_code STRING,
         |source STRING)
         |STORED BY 'carbondata'
         |TBLPROPERTIES(
         |'SORT_COLUMNS'='market_code, status, country_code, category_id, product_id, company_id',
         |'NO_INVERTED_INDEX'='name, company, release_date, artwork_url, slug, scrape_reviews, price, version, date_scraped, scrape_failed, sku, size, req_version, languages, created, support_url, license_url, scrape_review_delay, last_updated, bundle_id, bundle_product_count, family_sharing, purchased_separately_price, seller, required_devices, current_release_date, original_price, subtitle, promotional_text, editorial_badge_type, editorial_badge_name, installs, video_url, icon_url, banner_image_url, company_code, source',
         |'DICTIONARY_INCLUDE'='market_code,country_code',
         |'LONG_STRING_COLUMNS'='description, downloads_chart_url, permissions, whats_new, web_urls, related_apps, also_installed_apps, more_from_developer_apps, privacy_policy_url, publisher_email, ss_urls, link_apps, content_rating, requirements, app_store_notes',
         |'SORT_SCOPE'='LOCAL_SORT',
         |'CACHE_LEVEL'='BLOCKLET',
         |'TABLE_BLOCKSIZE'='256')
       """.stripMargin)

    carbon.sql(
      s"""
         |CREATE DATAMAP lucene_for_name_company_v2
         |ON TABLE productv1
         |USING 'lucene'
         |DMPROPERTIES ('INDEX_COLUMNS' = 'subtitle, require_os', 'SPLIT_BLOCKLET'='true', 'FLUSH_CACHE'='1g')
       """.stripMargin)

    carbon.sql(
      s"""
         |CREATE DATAMAP bloom_for_class_v1
         |ON TABLE productv1
         |USING 'bloomfilter'
         |DMPROPERTIES ('INDEX_COLUMNS' = 'class', 'BLOOM_SIZE'='640000', 'BLOOM_FPP'='0.00001', 'BLOOM_COMPRESS'='true')
       """.stripMargin)
  }



Collapse 
Message Input

Message insights-report-team

About
insights-report-team


Channel Details
 
Highlights
 
Pinned Items
 
12 Members
																						