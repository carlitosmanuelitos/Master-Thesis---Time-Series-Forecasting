
SELECT DISTINCT 
CONVERT(char(7), {o:creationTime}, 126) as 'Day',
{country.isocode} as 'Country',
{ct.code} as 'Default Category',
count({o:code}) AS 'Count',
sum({o.totalprice}) as 'TOTAL PRICE'

FROM {Order AS o
LEFT JOIN BaseStore AS store ON {o:store}={store:pk}
JOIN Country AS country ON {store:country} = {country:pk}
LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk}
LEFT JOIN OrderEntry AS oe ON {oe:order}={o:pk}
LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
LEFT JOIN Category as ct on {prd.defaultCategory} = {ct.pk} 
}


WHERE {o:originalVersion} IS NULL AND {o:code} NOT LIKE 'BLIND%'
AND {o:creationTime} BETWEEN CAST('2024-01-01 00:00:00' AS DATETIME) AND CAST('2024-01-31 23:59:59' AS DATETIME)
AND {country.isocode} = 'GB'
AND {oc.code} = 'WEBSITE'
AND {ot.typeid} IN ('ZTA')
group by CONVERT(char(7), {o:creationTime}, 126), {country.isocode}, {ct.code}



Day	Country	Default Category	Count	TOTAL PRICE
2024-01	GB	all	27234	1427026.57000000
2024-01	GB	all-consumable	2553	160549.25000000
2024-01	GB	all-device	9233	508701.91000000
2024-01	GB	ht-accessories	58	4177.15000000
2024-01	GB	ht-consumables	5019	632619.89000000
2024-01	GB	iluma-accessories	182	14057.29000000
2024-01	GB	iluma-consumables	18256	2261123.91000000



SELECT DISTINCT 
CONVERT(char(7), {o:creationTime}, 126) as 'Day',
{o:code} AS 'Count',
{country.isocode} as 'Country',
{prd:name} AS 'PRODUCT_DESCRIPTION',   
{ct.code} as 'Default Category',
{o.totalprice} as 'TOTAL PRICE'

FROM {Order AS o
LEFT JOIN BaseStore AS store ON {o:store}={store:pk}
JOIN Country AS country ON {store:country} = {country:pk}
LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
LEFT JOIN OrderType AS ot ON {o:orderType}={ot.pk}
LEFT JOIN OrderEntry AS oe ON {oe:order}={o:pk}
LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
LEFT JOIN Category as ct on {prd.defaultCategory} = {ct.pk} 
}


WHERE {o:originalVersion} IS NULL AND {o:code} NOT LIKE 'BLIND%'
AND {o:creationTime} BETWEEN CAST('2024-01-01 00:00:00' AS DATETIME) AND CAST('2024-01-31 23:59:59' AS DATETIME)
AND {country.isocode} = 'GB'
AND {oc.code} = 'WEBSITE'
AND {ot.typeid} IN ('ZTA')



Day	Count	Country	PRODUCT_DESCRIPTION	Default Category	TOTAL PRICE
2024-01	1003805424	GB	TEREA SIENNA BUNDLE (10)	iluma-consumables	105.00000000
2024-01	1003805501	GB	IQOS ILUMA Kit Pebble Beige	all-device	55.00000000
2024-01	1003805501	GB	TEREA MAUVE WAVE PACK	all	55.00000000
2024-01	1003805501	GB	TEREA TEAK PACK	all	55.00000000
2024-01	1003805526	GB	TEREA RUSSET PACK	all	26.00000000
2024-01	1003805526	GB	TEREA SIENNA PACK	all	26.00000000
2024-01	1003805526	GB	TEREA TEAK PACK	all	26.00000000
2024-01	1003805592	GB	TEREA GREEN BUNDLE (10)	iluma-consumables	170.00000000
2024-01	1003805609	GB	IQOS ILUMA One Kit Moss Green	all-device	29.00000000
2024-01	1003805609	GB	TEREA BLUE PACK	all	29.00000000
