SELECT 
    CONVERT(char(7), {o:creationTime}, 126) as 'Day',
    {country.isocode} as 'Country',
    {ct.code} as 'Default Category',
    COUNT(DISTINCT {o:code}) AS 'Order Count',
    SUM(top.total_order_price) as 'Total Price'
FROM {Order AS o
    LEFT JOIN BaseStore AS store ON {o:store}={store:pk}
    JOIN Country AS country ON {store:country} = {country:pk}
    LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
    LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
    LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk}
    LEFT JOIN OrderEntry AS oe ON {oe:order}={o:pk}
    LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
    LEFT JOIN Category AS ct on {prd.defaultCategory} = {ct.pk}
    LEFT JOIN (
        SELECT 
            {o:pk} as order_pk,
            MAX({o.totalprice}) as total_order_price
        FROM {Order AS o
            LEFT JOIN BaseStore AS store ON {o:store}={store:pk}
            JOIN Country AS country ON {store:country} = {country:pk}
            LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
            LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
            LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk}
        }
        WHERE 
            {o:originalVersion} IS NULL 
            AND {o:code} NOT LIKE 'BLIND%'
            AND {o:creationTime} BETWEEN CAST('2024-01-01 00:00:00' AS DATETIME) AND CAST('2024-01-31 23:59:59' AS DATETIME)
            AND {country.isocode} = 'GB'
            AND {oc.code} = 'WEBSITE'
            AND {ot.typeid} IN ('ZTA')
        GROUP BY 
            {o:pk}
    ) AS top ON {o:pk} = top.order_pk
}
WHERE 
    {o:originalVersion} IS NULL 
    AND {o:code} NOT LIKE 'BLIND%'
    AND {o:creationTime} BETWEEN CAST('2024-01-01 00:00:00' AS DATETIME) AND CAST('2024-01-31 23:59:59' AS DATETIME)
    AND {country.isocode} = 'GB'
    AND {oc.code} = 'WEBSITE'
    AND {ot.typeid} IN ('ZTA')
GROUP BY 
    CONVERT(char(7), {o:creationTime}, 126), 
    {country.isocode}, 
    {ct.code}
ORDER BY 
    CONVERT(char(7), {o:creationTime}, 126), 
    {ct.code};



Exception message: missing '}' for '{' at 989 in '{Order AS o LEFT JOIN BaseStore AS store ON {o:store}={store:pk} JOIN Country AS country ON {store:country} = {country:pk} LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus} LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk} LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk} LEFT JOIN OrderEntry AS oe ON {oe:order}={o:pk} LEFT JOIN Product AS prd ON {oe:product}={prd:pk} LEFT JOIN Category AS ct on {prd.defaultCategory} = {ct.pk} LEFT JOIN ( SELECT {o:pk} as order_pk, MAX({o.totalprice}) as total_order_price FROM {Order AS o LEFT JOIN BaseStore AS store ON {o:store}={store:pk} JOIN Country AS country ON {store:country} = {country:pk} LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus} LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk} LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk} }'

