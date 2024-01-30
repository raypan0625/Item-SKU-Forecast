# Item-SKU-Forecast
* The sales forecast gives the actual B2C demand for the next rolling 12 months on pcs count, SKU count by merchant, first cost, and gross revenue.
* It is adjusted for multiple confounding factors including but not limited to seasonality, OOS situation, product age, promotional activities, holiday/sales event, and market trends.

## Description
   - **Stockey**: Each individual item's identification number. e.g. 311520
   - **SKU**: A product's identification number that may contain same or different Stockeys.
   - **Master SKU**: The SKU, even contain the same Stockeys, maybe different across channels. Master SKU is used internally to keep track of sales, and Master SKU will be the same for SKUs with same Stockey setups.
   - **SKU_stockey_sorted**: How a SKU is combined together using different stockeys, separated by "_" and with total pcs count with in "()". e.g. 311520_311521(2)
   - **Category1**: Indoor/Outdoor
   - **Category3-2**: e.g. Desk, Sofa, Barstool, etc.

## Calculations  
1)	Each SKU on each channel has their rolling 12-month unadjusted sales and OH days for each month. Here we consider sales from our warehouse only (non-AMZ DI or innetwork, non-liquidation accounts, non-castlegate).  
    - sales from our warehouse: We want to get consumer true demand, so the sales are calculated as B2C sales. 
 ![WeChat Screenshot_20240129220255](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/1ba24d54-b238-434c-a395-afe4e21ef67f)

2)	Adjust for missed sales due to OOS for each SKU on each channel based on CDF placement and OH days to get the monthly “Sales<sub>OOS</sub>”..

    - CDF: cumulative distribution function for the rolling 12-month sales of each SKU within different channels, grouping them into five categories (0-20%, 20-40%, 40-60%, 60-80%, 80-100%), and adjusting weekly sales with a       decreasing adjustment parameter
![WeChat Screenshot_20240129225115](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/f41da569-ce87-40f9-88c9-bd432044d179)
![WeChat Screenshot_20240129225457](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/9947a49c-b2c5-4377-8718-0c7a1b1c6a72)  
Repeat this step for all months and all SKU channels.

3)	Divide each Sales<sub>OOS</sub> by corresponding monthly seasonality, get Sales<sub>OOS_yearly</sub> based on each month for each SKU channel.
     - This is linked to [Seasonality-Calculation](https://github.com/raypan0625/Seasonality-Calculation)  
![WeChat Screenshot_20240130000254](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/54200258-4012-4983-be8e-b755708ef5f0)   
Repeat this step for all SKU channels in all months.

4)	Assign monthly forecast weight based on the EMA Method or Pushback Method.
	  - **EMA Method**: When at least one of the latest 3 months are at least 25% on hand, those SKU-merchant will be categorized as “normal”, and the Monthly Forecast Weight will be assigned as following:
![WeChat Screenshot_20240130000533](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/fed96d20-194d-4754-8233-d1c1bee84321)
    - **Pushback Method**: When the latest 3 months all have their OH Days less than 25%, those SKU-merchant will be categorized as “not normal”, and we try to push back and find the first consecutive 4/3/2/1 month with OH Days greater than 50%. The Monthly Forecast Weight will be assigned as following:
![WeChat Screenshot_20240130000606](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/f2409818-6d2e-4d1a-a243-271a13e66d56)
    - In case a particular SKU is not normal, but we are unable to find any month with OH Days greater than 50%, those SKU-merchant will be categorized as “other”, the weight will be assigned according to the EMA method
5)	Calculate each weighted monthly forecast sales and aggregate on SKU_stockey_sorted.
![WeChat Screenshot_20240130000720](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/aab9204b-56dd-4740-8d1b-11d2724acedc)
6)  Calculate Amazon sales from Amazon portal. (DirectImport(DI)/InnetWork(IN)/DirectFulfillment(DF))
    - Calculate item’s DI, IN, DF percentage.
        - If the item is sold on DI, then:
![WeChat Screenshot_20240130001307](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/97958231-49a9-4ba2-af5f-1e5530947a60)
        - If the item is not sold on DI, then:
![WeChat Screenshot_20240130001047](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/391cef04-d1b9-479b-bf5a-e33eade8940d)
        - If the item is only sold on DF, then:
![WeChat Screenshot_20240130001047](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/aca156bb-f544-42bc-b572-dd7c42bf98f6)

    - Get the monthly AMZ portal weighted sales using the EMA/Pushback Method OOS adjustment as shown above, and aggregate on SKU_stockey_sorted.
![WeChat Screenshot_20240130001432](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/18d4040b-c043-4f18-bec7-60a7271c2c8d)
7) Get the monthly AMZ portal weighted sales using the EMA/Pushback Method OOS adjustment as shown above, and aggregate on SKU_stockey_sorted.
8) Convert SKU_stockey_sorted level forecast to HM level.
    - Utilize the sku_hm_map to convert forecast from SKU_stockey to Stockey (HM_stockey)
![WeChat Screenshot_20240130001608](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/1f297972-f9ff-4c22-b1e2-ba8e39e714ed)

![WeChat Screenshot_20240130001706](https://github.com/raypan0625/Item-SKU-Forecast/assets/103529023/f7e4007c-6b10-4a7d-bd7e-6684080b53e9)


## Other Note
### Amazon portal sales difference
- DI:
- IN:
- DF:

## Author

Yulin Pan raypan0625@gmail.com

## Version History

* 0.2
    * added rolling seasonality calculation with category specialization
* 0.1
    * Initial Release

