# Adventure Works Report

<a href = "../Adventure%20Works%20Report/resources/ExecDashboard.png">
<img alt="Executive Page of AdventureWorks Dashboard" width="400px" src="../Adventure%20Works%20Report/resources/ExecDashboard.png">
</a>

[Link to Hosted Report](https://app.powerbi.com/view?r=eyJrIjoiN2ZhNTAzM2YtNmJjNS00M2RjLWJlMmEtMWFiNzEwYjRmOTlkIiwidCI6IjlhYzIwMTdiLTBiNjAtNDgzZC1iYjkzLWVjOWU3YTU1MDE3MCJ9)

## Executive Summary

Developed an interactive Power BI sales analytics report using the Adventure Works dataset to track revenue, profit, returns, regional performance, product trends, and customer segments. The project demonstrates Power Query data preparation, star schema modeling, DAX measures, executive dashboard design, and in-report documentation.

## Situation
I was tasked with a project to analyze raw sales and operational data for AdventureWorks, a fictional global manufacturing company specializing in cycling equipment. The project's starting point was a collection of disparate CSV files, and there was no existing centralized tool to monitor key performance indicators (KPIs) or make informed, data-driven decisions.

## Task
My primary objective was to develop a comprehensive Power BI dashboard to track critical KPIs (sales, revenue, profit, and returns), compare performance across different sales regions, analyze trends at the product level, and identify high-value customers. The goal was to transform the raw data into a cohesive, interactive, and actionable business intelligence tool.

## Action
1. **Data Transformation & Modeling:** I connected to multiple raw CSV files within Power BI, including data on transactions, returns, products, customers, and sales territories. Using Power Query, I cleaned, transformed, and shaped the data to ensure accuracy and consistency. I then built a relational star-schema data model, creating relationships between the fact and dimension tables to enable robust analysis.

2. **DAX Calculations:** I developed several key measures and calculated columns using Data Analysis Expressions (DAX). These calculations formed the analytical backbone of the report, allowing for the dynamic calculation of metrics such as Total Revenue, Profit Margin, Return Rate, and Year-over-Year Sales Growth.

3. **Dashboard Design & Visualization:** I designed a multi-page, interactive dashboard focused on user experience. The main page provided a high-level overview of key KPIs. Subsequent pages offered deep-dive analyses into regional performance, product-level trends, and customer segmentation. I utilized a variety of visualizations, including bar charts, line graphs, maps, and slicers, to present the data in an intuitive and compelling way.

4. **Data Dictionary & Documentation:** To enhance usability and transparency, I created a dynamic data dictionary page within the dashboard. By leveraging DAX INFO functions, I exposed the metadata and descriptions for all tables and measures used in the model, providing clear, in-report documentation for both business users and technical stakeholders.

## Result
The project culminated in a dynamic and fully interactive Power BI dashboard that successfully visualized the AdventureWorks dataset. The dashboard delivered on all project objectives, enabling users to:
- Track and monitor sales, revenue, and profit in near real-time.
- Easily compare performance between different sales territories, identifying top-performing and underperforming regions.
- Drill down into product-level data to understand sales trends and identify popular items.
- Segment and identify high-value customers, creating opportunities for targeted marketing and sales strategies.

This analysis transformed raw data into actionable intelligence, providing a clear and comprehensive tool for data exploration and strategic insights.

<a href = "../Adventure%20Works%20Report/resources/Map.png">
<img align="left" alt="Map Page of AdventureWorks Dashboard" width="250px" src="../Adventure%20Works%20Report/resources/Map.png">
</a>
<a href = "../Adventure%20Works%20Report/resources/ProductDetail.png">
<img align="left" alt="Product Detail Page of AdventureWorks Dashboard" width="250px" src="../Adventure%20Works%20Report/resources/ProductDetail.png">
</a>
<a href = "../Adventure%20Works%20Report/resources/CustomerDetail.png">
<img align="left" alt="Customer Detail Page of AdventureWorks Dashboard" width="250px" src="../Adventure%20Works%20Report/resources/CustomerDetail.png">
</a>
<a href = "../Adventure%20Works%20Report/resources/DataDictionary.png">
<img align="left" alt="Data Dictionary Page of AdventureWorks Dashboard" width="250px" src="../Adventure%20Works%20Report/resources/DataDictionary.png">
</a>