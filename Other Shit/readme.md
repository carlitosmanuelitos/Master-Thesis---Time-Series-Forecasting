QUERY 1 DE: 

SELECT
       x.ORDER_NUMBER,
       x.CHANNEL,
       x.ORDER_TYPE,
       x.DATE,
       x.ORDER_ENTRY_AMOUNT_GROSS,
       x.ORDER_ENTRY_VAT,
       x.ORDER_ENTRY_DISCOUNT_NET,
       x.VOUCHER_ID,
       x.PLACED_BY,
       x.ENTRY_GLOBAL_CODE,
       x.ENTRY_DESCRIPTION,
       x.ENTRY_QUANTITY,
       x.LOGICAL_CHANNEL,
       x.PROMOTION_NAME,
       x.PROMOTION_DESCRIPTION,
       x.DISCOUNT_TYPE,
       x.ORDER_AMOUNT_GROSS,
       x.ORDER_VAT
FROM (
{{
SELECT DISTINCT
      {o:code} AS 'ORDER_NUMBER',
       {o:totalprice} AS 'ORDER_AMOUNT_GROSS',
       {o:pmiVat} AS 'ORDER_VAT',
       {o:date} AS 'DATE',
       {ot:typeCode} AS 'ORDER_TYPE',
       {CR.couponcode} AS 'VOUCHER_ID',
       {emp:provisioningexternalid} AS 'PLACED_BY',
       {oc:code} AS 'CHANNEL',
       {oe:totalprice} AS 'ORDER_ENTRY_AMOUNT_GROSS',
 	   ({oe:baseprice} * ({oe:quantity})) - {oe:totalprice}  AS 'ORDER_ENTRY_DISCOUNT_NET',
       {oe:pmivatvalue} AS 'ORDER_ENTRY_VAT',
       {oe:quantity}  AS 'ENTRY_QUANTITY',
       {prd:code} AS 'ENTRY_GLOBAL_CODE',
       {prd:name} AS 'ENTRY_DESCRIPTION',
       {sale:code} AS 'LOGICAL_CHANNEL',
       {rbp:name} AS 'PROMOTION_NAME',
       {rbp:promotiondescription[en]} AS 'PROMOTION_DESCRIPTION',
       'LINE ITEM'  AS 'DISCOUNT_TYPE'
  
FROM {Order AS o
    JOIN Orderentry AS oe ON {oe:order} = {o:pk}
    LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
    LEFT JOIN OrderType AS ot ON {o:orderType}={ot:pk}
    LEFT JOIN SalesApplication AS sale ON {sale:pk}={o:salesApplication}
    LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
    LEFT JOIN CouponRedemption AS CR ON {CR.order}={o.pk}
    LEFT JOIN Employee as emp on {o:placedby}={emp:pk}
    LEFT JOIN CmsSite AS site ON {o:site}={site:pk}
    LEFT JOIN BaseStore AS store ON {o:store}={store:pk} AND {store:uid} LIKE '%-de-%'
    JOIN Country AS country ON {store:country} = {country:pk}
    JOIN PromotionResult AS pr ON {pr:order} = {o:pk}
    JOIN RuleBasedPromotion AS rbp ON {pr:promotion} = {rbp:pk}
    JOIN RuleBasedOrderEntryAdjustAction as pb
    ON {pb:promotionresult} = {pr:pk}
    AND {pb:orderentryproduct} = {oe:product}
    AND {pb:orderentrynumber} = {oe:entrynumber}
    AND {pb:markedapplied} = 1
    AND {pb:guid} = SUBSTRING({oe.discountvaluesinternal},6,44)
}
  
WHERE {o:originalVersion} IS NULL
AND {o:versionID} is null
AND {o:status} NOT IN ('8796125691995')
AND {country.isocode}='DE'
AND {o:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)
}}
UNION ALL
{{
SELECT DISTINCT
       {o:code} AS 'ORDER_NUMBER',
       {o:totalprice} AS 'ORDER_AMOUNT_GROSS',
       {o:pmiVat} AS 'ORDER_VAT',
       {o:date} AS 'DATE',
       {ot:typeCode} AS 'ORDER_TYPE',
       {CR.couponcode} AS 'VOUCHER_ID',
       {emp:provisioningexternalid} AS 'PLACED_BY',
       {oc:code} AS 'CHANNEL',
       {oe:totalprice} AS 'ORDER_ENTRY_AMOUNT_GROSS',
       ({oe:baseprice} * ({oe:quantity})) - {oe:totalprice}  AS 'ORDER_ENTRY_DISCOUNT_NET',
       {oe:pmivatvalue} AS 'ORDER_ENTRY_VAT',
       {oe:quantity}  AS 'ENTRY_QUANTITY',
       {prd:code} AS 'ENTRY_GLOBAL_CODE',
       {prd:name} AS 'ENTRY_DESCRIPTION',
       {sale:code} AS 'LOGICAL_CHANNEL',
       {rbp:name} AS 'PROMOTION_NAME',
       {rbp:promotiondescription[en]} AS 'PROMOTION_DESCRIPTION',
       'LINE ITEM'  AS 'DISCOUNT_TYPE'
  
FROM {Order AS o
JOIN Orderentry AS oe ON {oe:order} = {o:pk}
LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
LEFT JOIN OrderType AS ot ON {o:orderType}={ot:pk}
LEFT JOIN OrderStatus AS hst ON {o:status}={hst:pk}
LEFT JOIN CouponRedemption AS CR ON {CR.order}={o.pk}
LEFT JOIN Employee as emp on {o:placedby}={emp:pk}
LEFT JOIN CmsSite AS site ON {o:site}={site:pk}
LEFT JOIN BaseStore AS store ON {o:store}={store:pk} AND {store:uid} LIKE '%-de-%'
JOIN Country AS country ON {store:country} = {country:pk}
LEFT JOIN SalesApplication AS sale ON {sale:pk}={o:salesApplication}
LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
LEFT JOIN OrderReason AS or ON {o:OrderReason} = {or:pk}
JOIN PromotionResult AS pr ON {pr:order} = {o:pk}
JOIN RuleBasedPromotion AS rbp ON {pr:promotion} = {rbp:pk}
}
WHERE {o:originalVersion} IS NULL
AND {o:versionID} is null
AND {o:status} NOT IN ('8796125691995')
AND {country.isocode}='DE'
AND {o:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)


AND NOT EXISTS ({{
                   SELECT {pb2:pk}
                      FROM {RuleBasedOrderEntryAdjustAction as pb2 JOIN PromotionResult AS pr ON {pb2:promotionresult} = {pr:pk}
                              JOIN Order AS o2 ON {pr:order} = {o2:pk}
                              JOIN Orderentry AS oe2 ON {oe2:order} = {o2:pk}
                              }
                      WHERE  {pb2:promotionresult} = {pr:pk}
                        AND {pb2:orderentryproduct} = {oe2:product}
                        AND {pb2:orderentrynumber} = {oe2:entrynumber}
                        AND {pb2:markedapplied} = 1
                        AND {pb2:guid} =  SUBSTRING({oe.discountvaluesinternal},6,44)
                        AND {oe2:pk} = {oe:pk} 
                        AND {o2:pk} = {o:pk}
               			AND {o:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)
                    }})
AND NOT EXISTS ({{
                   SELECT {pb2:pk}
                      FROM {RuleBasedOrderAdjustTotalAction as pb2 JOIN PromotionResult AS pr ON {pb2:promotionresult} = {pr:pk}
                              JOIN Order AS o2 ON {pr:order} = {o2:pk}
                              }
                      WHERE  {pb2:promotionresult} = {pr:pk}
                        AND {pb2:markedapplied} = 1
                        AND {pb2:guid} =  SUBSTRING({oe.discountvaluesinternal},6,44)
                        AND {o2:pk} = {o:pk}
               			AND {o:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)
                    }})
}}) x




QUERY 2: 

SELECT
       x.ORDER_NUMBER,
       x.CHANNEL,
       x.ORDER_TYPE,
       x.ORDER_REASON_ID,
       x.ORDER_REASON_CODE,
       x.DATE,
       x.ORDER_ENTRY_AMOUNT_GROSS,
       x.ORDER_ENTRY_DISCOUNT_NET,
       x.ORDER_ENTRY_VAT,
       x.ORDER_ENTRY_NET_Without_Tax_WithoutDiscount,
       x.ENTRY_GLOBAL_CODE,
       x.ENTRY_DESCRIPTION,
       x.ENTRY_QUANTITY,
       x.LOGICAL_CHANNEL,
       x.PROMOTION_NAME,
       x.PROMOTION_ID,
       x.DISCOUNT_TYPE,
       x.ORDER_AMOUNT_GROSS,
       x.ORDER_VAT,
       x.ORDER_AMOUNT_NET
FROM (
{{
SELECT DISTINCT
      {o:code} AS 'ORDER_NUMBER',
       {oc:code} AS 'CHANNEL',
       {ot:typeCode} AS 'ORDER_TYPE',
       {or:reasonId} AS 'ORDER_REASON_ID',
       {or:reasonCode} AS 'ORDER_REASON_CODE',
       {o:date} AS 'DATE',
       {oe:totalprice} AS 'ORDER_ENTRY_AMOUNT_GROSS',
       ({oe:baseprice} * ({oe:quantity})) - {oe:totalprice}  AS 'ORDER_ENTRY_DISCOUNT_NET',
       {oe:pmivatvalue} AS 'ORDER_ENTRY_VAT',
       ({oe:totalprice}-(({oe:baseprice} * ({oe:quantity})) - {oe:totalprice})-{oe:pmivatvalue}) AS 'ORDER_ENTRY_NET_Without_Tax_WithoutDiscount',
       {prd:code} AS 'ENTRY_GLOBAL_CODE',
       {prd:name} AS 'ENTRY_DESCRIPTION',
       {oe:quantity}  AS 'ENTRY_QUANTITY',
       {sale:code} AS 'LOGICAL_CHANNEL',
       {rbp:name} AS 'PROMOTION_NAME',
       {rbp:code} AS 'PROMOTION_ID',
       'LINE ITEM'  AS 'DISCOUNT_TYPE',
       {o:totalprice} AS 'ORDER_AMOUNT_GROSS',
       {o:pmiVat} AS 'ORDER_VAT',
       ({o:totalprice}-{o:pmiVat}) AS 'ORDER_AMOUNT_NET',
       {country:isocode} AS 'country',
       {pmist:code} AS 'PMI_ORDER_STATUS'
  
FROM {Order AS o
JOIN Orderentry AS oe ON {oe:order} = {o:pk}
LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
LEFT JOIN CmsSite AS site ON {o:site}={site:pk}
LEFT JOIN BaseStore AS store ON {o:store}={store:pk} AND {store:uid} LIKE '%-de-%'
JOIN Country AS country ON {store:country} = {country:pk}
LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
LEFT JOIN OrderType AS ot ON {o:orderType}={ot:pk}
LEFT JOIN OrderStatus AS hst ON {o:status}={hst:pk}
LEFT JOIN SalesApplication AS sale ON {sale:pk}={o:salesApplication}
LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
LEFT JOIN OrderReason AS or ON {o:OrderReason} = {or:pk}
JOIN PromotionResult AS pr ON {pr:order} = {o:pk}
JOIN RuleBasedPromotion AS rbp ON {pr:promotion} = {rbp:pk}
JOIN RuleBasedOrderEntryAdjustAction as pb ON {pb:promotionresult} = {pr:pk} AND {pb:orderentryproduct} = {oe:product}
AND {pb:orderentrynumber} = {oe:entrynumber} AND {pb:markedapplied} = 1
AND {pb:guid} = SUBSTRING(SUBSTRING({oe.discountvaluesinternal},6,49),1,44)
}
WHERE {o:originalVersion} IS NULL AND {o:versionID} is null
AND {pmist:code} NOT IN ('CANCELLED') AND {country.isocode}='DE'
AND {o:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)
}}
UNION ALL
{{
SELECT DISTINCT
       {o:code} AS 'ORDER_NUMBER',
       {oc:code} AS 'CHANNEL',
       {ot:typeCode} AS 'ORDER_TYPE',
       {or:reasonId} AS 'ORDER_REASON_ID',
       {or:reasonCode} AS 'ORDER_REASON_CODE',
       {o:date} AS 'DATE',
       {oe:totalprice} AS 'ORDER_ENTRY_AMOUNT_GROSS',
       ( {oe:baseprice} * ({oe:quantity}) ) - {oe:totalprice}  AS 'ORDER_ENTRY_DISCOUNT_NET',
       {oe:pmivatvalue} AS 'ORDER_ENTRY_VAT',
       ({oe:totalprice}-(({oe:baseprice} * ({oe:quantity})) - {oe:totalprice})-{oe:pmivatvalue}) AS 'ORDER_ENTRY_NET_Without_Tax_WithoutDiscount',
       {prd:code} AS 'ENTRY_GLOBAL_CODE',
       {prd:name} AS 'ENTRY_DESCRIPTION',
       {oe:quantity}  AS 'ENTRY_QUANTITY',
       {sale:code} AS 'LOGICAL_CHANNEL',
       NULL AS 'PROMOTION_NAME',
       NULL AS 'PROMOTION_ID',
       'LINE ITEM'  AS 'DISCOUNT_TYPE',
       {o:totalprice} AS 'ORDER_AMOUNT_GROSS',
       {o:pmiVat} AS 'ORDER_VAT',
       ({o:totalprice}-{o:pmiVat}) AS 'ORDER_AMOUNT_NET',
       {country:isocode} AS 'country',
       {pmist:code} AS 'PMI_ORDER_STATUS'
  
FROM {Order AS o
JOIN Orderentry AS oe ON {oe:order} = {o:pk}
LEFT JOIN Product AS prd ON {oe:product}={prd:pk}
LEFT JOIN CmsSite AS site ON {o:site}={site:pk}
LEFT JOIN BaseStore AS store ON {o:store}={store:pk} AND {store:uid} LIKE '%-de-%'
JOIN Country AS country ON {store:country} = {country:pk}
LEFT JOIN PmiOrderStatus AS pmist ON {pmist:pk}={o:pmiOrderStatus}
LEFT JOIN OrderType AS ot ON {o:orderType}={ot:pk}
LEFT JOIN OrderStatus AS hst ON {o:status}={hst:pk}
LEFT JOIN SalesApplication AS sale ON {sale:pk}={o:salesApplication}
LEFT JOIN OrderChannel AS oc ON {o:orderChannel} = {oc:pk}
LEFT JOIN OrderReason AS or ON {o:OrderReason} = {or:pk}
JOIN PromotionResult AS pr ON {pr:order} = {o:pk}
JOIN RuleBasedPromotion AS rbp ON {pr:promotion} = {rbp:pk}
}
  
WHERE {o:originalVersion} IS NULL AND {o:versionID} is null
AND {pmist:code} NOT IN ('CANCELLED') AND {country.isocode}='DE'
AND {o:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)



AND NOT EXISTS ({{
                   SELECT {pb2:pk}
                      FROM {RuleBasedOrderEntryAdjustAction as pb2 JOIN PromotionResult AS pr ON {pb2:promotionresult} = {pr:pk}
                              JOIN Order AS o2 ON {pr:order} = {o2:pk}
                              JOIN Orderentry AS oe2 ON {oe2:order} = {o2:pk}
                              }
                      WHERE  {pb2:promotionresult} = {pr:pk}
                        AND {pb2:orderentryproduct} = {oe2:product}
                        AND {pb2:orderentrynumber} = {oe2:entrynumber}
                        AND {pb2:markedapplied} = 1
                        AND {pb2:guid} = SUBSTRING(SUBSTRING({oe2.discountvaluesinternal},6,49),1,44)
                        AND {oe2:pk} = {oe:pk}  
                        AND {o2:pk} = {o:pk} 
                		    AND {o2:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)
                    }})
AND NOT EXISTS ({{
                   SELECT {pb2:pk}
                      FROM {RuleBasedOrderAdjustTotalAction as pb2 JOIN PromotionResult AS pr ON {pb2:promotionresult} = {pr:pk}
                                JOIN Order AS o2 ON {pr:order} = {o2:pk}
                              }
                      WHERE  {pb2:promotionresult} = {pr:pk}
                        AND {pb2:markedapplied} = 1
                        AND {pb2:guid} = SUBSTRING(SUBSTRING({o2.globaldiscountvaluesinternal},6,49),1,44)
                        AND {o2:pk} = {o:pk}    
                		    AND {o2:creationTime} BETWEEN CAST('2024-06-03 12:19:00' AS DATETIME) AND CAST('2024-06-03 12:20:00' AS DATETIME)
                    }})  
}}) x
