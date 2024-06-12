SELECT 
    x.Day,
    x.Country,
    x.Default_Category,
    COUNT(DISTINCT x.Order_Code) AS Order_Count,
    SUM(x.Total_Order_Price) AS Total_Price
FROM (
    SELECT DISTINCT
        CONVERT(char(7), {o:creationTime}, 126) as 'Day',
        {o:code} as 'Order_Code',
        {country.isocode} as 'Country',
        {ct.code} as 'Default_Category',
        {o.totalprice} as 'Total_Order_Price'
    FROM {Order AS o
        LEFT JOIN BaseStore AS store ON {o:store}={store:pk}
        JOIN Country AS country ON {store:country} = {country:pk}
        LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
        LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
        LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk}
        LEFT JOIN OrderEntry AS oe ON {oe:order}={o:pk}
        LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
        LEFT JOIN Category AS ct ON {prd:defaultCategory} = {ct:pk}
    }
    WHERE 
        {o:originalVersion} IS NULL 
        AND {o:code} NOT LIKE 'BLIND%'
        AND {o:creationTime} BETWEEN CAST('2024-01-01 00:00:00' AS DATETIME) AND CAST('2024-01-31 23:59:59' AS DATETIME)
        AND {country.isocode} = 'GB'
        AND {oc.code} = 'WEBSITE'
        AND {ot.typeid} IN ('ZTA')
) x
GROUP BY 
    x.Day, 
    x.Country, 
    x.Default_Category
ORDER BY 
    x.Day, 
    x.Default_Category;
