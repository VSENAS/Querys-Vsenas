# Querys-Vsenas
Only for use of Team Data Marketplace
GMV 1 -- DENISE
(FEITO)
Faturamento Comissão -- DENISE
-- -----------------------------------------------------------------------------------------------------------------------------------------
quantidade de pedidos por mes (FEITO) azul bem claro 

select count (DISTINCT (pedido.ORDER_ID)), EXTRACT(month from pedido.PURCHASE_DATE) from AC_ADMIN.ECAD_ORDER PEDIDO
WHERE pedido.STORE_ID = 'NCMP'
AND PURCHASE_DATE >= to_date('01-JAN-2018','DD-MON-YY')
GROUP BY EXTRACT(month from pedido.PURCHASE_DATE) order by EXTRACT(month from pedido.PURCHASE_DATE);


-- -----------------------------------------------------------------------------------------------------------------------------------------
SKUS DISTINTOS VENDIDOS (FEITO) laranja

-- 1 quantidade de skus distintos vendidos ao longo dos meses FEITO

select count (DISTINCT (pedidoline.SKU_ID)), EXTRACT(month from pedido.PURCHASE_DATE) from AC_ADMIN.ECAD_ORDER PEDIDO
   inner join AC_ADMIN.ECAD_DELIVERY ENTREGA on ENTREGA.ORDER_ID = PEDIDO.ORDER_ID
   inner join AC_ADMIN.ECAD_ORDER_LINE PEDIDOLINE on PEDIDOLINE.DELIVERY_ID = ENTREGA.DELIVERY_ID
WHERE pedido.STORE_ID = 'NCMP'  AND entrega.STORE_ID = 'NCMP' AND PEDIDOLINE.STORE_ID = 'NCMP'
AND PURCHASE_DATE >= to_date('01-JAN-2018','DD-MON-YY')


-- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
TOTAL DE LOJISTAS ATIVOS (FEITO)

-- COMENTÁRIOS

/*select * from AC_ADMIN.ECMA_SKU_MP_RELATED_SKU_SELLER where REFERENCEID = '11520181' and STORE_ID ='NCMP' and STORE_QUALIFIER_ID = '10612';


       -- REFERENCEID = '11520181' and
                           -- (ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID = '10138' or ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID = '31644'
                           --   OR ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID = '17767'
                           --    )
                           --  ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID = '10612' and */


select * from AC_ADMIN.ECMA_SKU_MP_RELATED_SKU_SELLER
    inner join AC_ADMIN.ECAD_WAREHOUSE on ECAD_WAREHOUSE.STORE_QUALIFIER_ID = ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID
    inner join AC_ADMIN.ECAD_CURRENT_STOCK on (ECAD_CURRENT_STOCK.WAREHOUSE_ID = ECAD_WAREHOUSE.WAREHOUSE_ID AND ECMA_SKU_MP_RELATED_SKU_SELLER.SKU_ID = ECAD_CURRENT_STOCK.SKU_ID )
    inner join AC_ADMIN.ECAD_STORE_QUALIFIER store on store.STORE_QUALIFIER_ID = ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID
    where REFERENCEID = '11520181' and
          ECAD_WAREHOUSE.STORE_ID ='NCMP' and
          ECAD_CURRENT_STOCK.STORE_ID ='NCMP' AND
          store.STORE_ID ='NCMP' AND
          ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_ID ='NCMP' AND
          ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID = '10612' AND
          store.ACTIVE = 'Y' AND
          COMERCIAL_STATUS = '1';

/*-- O QUE DEFINIU A DIFERENÇA ENTRE BANCO E SITE FOI O STORE_QUALIFIER ACTIVE = 'Y' ALGUMAS LOJA MESMO ESTANDO NO SITE,
COM OFERTA DE SKU ATIVA E COM ESTOQUE... APARENCEM INATIVOS NA STORE_QUALIFIER*/

-- contando lojistas
select distinct STORE_QUALIFIER_ID from (select * from AC_ADMIN.ECMA_SKU_MP_RELATED_SKU_SELLER
    inner join AC_ADMIN.ECAD_WAREHOUSE on ECAD_WAREHOUSE.STORE_QUALIFIER_ID = ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID
    inner join AC_ADMIN.ECAD_CURRENT_STOCK on (ECAD_CURRENT_STOCK.WAREHOUSE_ID = ECAD_WAREHOUSE.WAREHOUSE_ID AND ECMA_SKU_MP_RELATED_SKU_SELLER.SKU_ID = ECAD_CURRENT_STOCK.SKU_ID )
    inner join AC_ADMIN.ECAD_STORE_QUALIFIER store on store.STORE_QUALIFIER_ID = ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID
    where
        --REFERENCEID = '11520181' and
          ECAD_WAREHOUSE.STORE_ID ='NCMP' and
          ECAD_CURRENT_STOCK.STORE_ID ='NCMP' AND
          store.STORE_ID ='NCMP' AND
          ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_ID ='NCMP' AND
        --  ECMA_SKU_MP_RELATED_SKU_SELLER.STORE_QUALIFIER_ID = '10612' AND
          store.ACTIVE = 'Y' AND
          COMERCIAL_STATUS = '1');
---------------------------------------------------
CATALOGO
Tempo de cadastro (FEITO)

/*Tempo de cadastro.
Como product owner responsável pela catalogação de produtos MarketPlace
Eu quero saber qual é o tempo de cadastro de um produto
Para, a primeiro momento, verificar se o projeto de associação automática por EAN diminuiu esse tempo de cadastro */

select count(1), trunc(Data_Integracao, 'DD'), Origem from (
select * from (
select
    case
        when p.NAM_APPROVER is not null then 'Tela de Aprovação'
        when p.DEDUPLICATION_USER is not null then 'Tela de desduplicação'
        else 'Coluna CY'
    end Origem,
    case
        when p.NAM_APPROVER is not null then DAT_APPROVAL
        when p.DEDUPLICATION_USER is not null then DEDUPLICATION_DATE
        else DT_CREATED
    end Data_Integracao,
    sku_id, skuidorigin, p.id_importer_info, i.dt_created Data_de_chegada
from ac_admin.ecia_imported_product p inner join ac_admin.ecia_importer_info i
on i.id_importer_info = p.id_importer_info
where p.id_product_status = 1
and p.id_deny_reason is null)
where data_integracao > Data_de_chegada
  and Data_de_chegada between '01/01/2018' and current_date
)
group by  Origem, trunc(Data_Integracao, 'DD')
order by Origem
;

-----------------------------------------------------------------------------------

--------------------------------------------------------------------------------------------------------
--TEMPO 1° ESTOQUE E PRIMEIRA VENDA (FEITO) Preto


SELECT DISTINCT SKU.SKU_ID, sku.INS_DATE , (select min(PURCHASE_DATE) from AC_ADMIN.ECAD_ORDER PEDIDO
   inner join AC_ADMIN.ECAD_DELIVERY ENTREGA on ENTREGA.ORDER_ID = PEDIDO.ORDER_ID
   inner join AC_ADMIN.ECAD_ORDER_LINE PEDIDOLINE on PEDIDOLINE.DELIVERY_ID = ENTREGA.DELIVERY_ID
   where PEDIDOLINE.SKU_ID = sku.SKU_ID AND pedido.STORE_ID = 'NCMP'
     AND ENTREGA.STORE_ID = 'NCMP' AND PEDIDOLINE.STORE_ID = 'NCMP'

                                     ) AS primeiraVenda,
                (TO_DATE((min(PURCHASE_DATE)), 'dd/mm/yyyy') - TO_DATE((min(SKU.INS_DATE)),'dd/mm/yyyy')) AS DIFF,
                (

select min(INS_DATE) from AC_ADMIN.ECAD_STOCK_HISTORY where SKU_ID = SKU.SKU_ID) as primeiroestoque

-- a diferença entre ins_date de inserção (ecad_sku) - ins_date de inserção (ecad_stock_history)  é o tempo de primeiro estoque

FROM AC_ADMIN.ECAD_SKU SKU
INNER JOIN AC_ADMIN.ECAD_ORDER_LINE ON ECAD_ORDER_LINE.SKU_ID = SKU.SKU_ID
INNER JOIN AC_ADMIN.ECAD_DELIVERY ON ECAD_ORDER_LINE.DELIVERY_ID = ECAD_DELIVERY.DELIVERY_ID
INNER JOIN AC_ADMIN.ECAD_ORDER ON ECAD_DELIVERY.ORDER_ID = ECAD_ORDER.ORDER_ID
WHERE PURCHASE_DATE IS NOT NULL
AND SKU.INS_DATE >= to_date('01-ABR-2018','DD-MON-YY')
AND PURCHASE_DATE >= to_date('01-ABR-2018','DD-MON-YY')
AND SHOP_NAME NOT LIKE '%Extra%'
AND SHOP_NAME NOT LIKE '%EXTRA%'
AND SHOP_NAME NOT LIKE '%PF%'
AND SHOP_NAME NOT LIKE '%PONTOFRIO%'
AND SHOP_NAME NOT LIKE '%PontoFrio%'
AND SHOP_NAME NOT LIKE '%Ponto-Frio%'
AND SHOP_NAME NOT LIKE '%PONTO-FRIO%'
AND SHOP_NAME NOT LIKE '%Ponto Frio%'
AND SHOP_NAME NOT LIKE '%PONTO FRIO%'
AND SHOP_NAME NOT LIKE '%CB%'
AND SHOP_NAME NOT LIKE '%CB-%'
AND SHOP_NAME NOT LIKE '%CASASBAHIA%'
AND SHOP_NAME NOT LIKE '%CASAS-BAHIA%'
AND SHOP_NAME NOT LIKE '%CASAS BAHIA%'
AND SHOP_NAME NOT LIKE '%CasasBahia%'
AND SHOP_NAME NOT LIKE '%Casas Bahia%'
AND SHOP_NAME NOT LIKE '%Casas-Bahia%'
AND SHOP_NAME NOT LIKE '%Casas - Bahia%'
AND SHOP_NAME NOT LIKE '%BLACKFRIDAY%'
AND SHOP_NAME NOT LIKE '%BLACK FRIDAY%'
AND SHOP_NAME NOT LIKE '%BLACK-FRIDAY%'
AND SHOP_NAME NOT LIKE '%black friday%'
AND SHOP_NAME NOT LIKE '%Black Friday%'
AND SHOP_NAME NOT LIKE '%Black-Friday%'
AND SHOP_NAME NOT LIKE '%BlackFriday%'
AND SKU.STORE_ID = 'NCMP'
GROUP BY SKU.SKU_ID,
         SKU.INS_DATE
ORDER BY SKU.INS_DATE ASC
;


-- -------------------------------------------------------------------------------------------------------
QTDE de skus por canal de entrada (FEITO) CINZA

 select distinct(ECMA_SKU_IMPORTER_ORIGIN.store_qualifier_id) as ID_LOJA, 
upper(store_qualifier_name) AS NOME_LOJA, 
CASE ECMA_SKU_IMPORTER_ORIGIN.store_qualifier_id
    WHEN '12231' 
      THEN count(ecma_sku_importer_origin.sku_origin_id)+32781
    ELSE
       count(ecma_sku_importer_origin.sku_origin_id)
    END AS qtd
from AC_ADMIN.ECMA_SKU_IMPORTER_ORIGIN 
INNER JOIN ac_admin.ecad_store_qualifier on (ecad_store_qualifier.store_qualifier_id = ECMA_SKU_IMPORTER_ORIGIN.store_qualifier_id)
where SOURCE_SYSTEM_TP = 'IMPORTER_V4_API'
AND ECMA_SKU_IMPORTER_ORIGIN.store_qualifier_id NOT IN (3, 12396, 11983, 13584, 31102)
group by ECMA_SKU_IMPORTER_ORIGIN.store_qualifier_id, store_qualifier_name
order by qtd desc;
-- ------------------------------------------------------------------------------------------------------
pedidos por categoria

select distinct count(order_id), CATEGORY_NAME, O.PURCHASE_DATE from AC_ADMIN.ECAD_ORDER O
inner join AC_ADMIN.ECAD_WAREHOUSE on ECAD_WAREHOUSE.STORE_QUALIFIER_ID = O.SHOP_CODE
inner join AC_ADMIN.ECIA_CATEGORY on O.SHOP_CODE = ECIA_CATEGORY.SHOP_CODE
inner join AC_ADMIN.ECAD_NONMARKET_STRUCTURE CATEGORIA on LEVEL_ID = ID_M_CATEGORY
inner join AC_ADMIN.ECIA_CATEGORY_TREE_MVW cat on CATEGORIA.LEVEL_ID = CAT.id
--inner join AC_ADMIN.ECIA_CATEGORY c on c.ID_M_CATEGORY = cat.ID
--inner join AC_ADMIN.ECAD_LEVEL l on l.LEVEL_ID = CATEGORIA.LEVEL_ID
where ECAD_WAREHOUSE.STORE_ID = 'NCMP' and
  PURCHASE_DATE between  '01/01/2018' and current_date
  group by CATEGORY_NAME,PURCHASE_DATE;

-- -------------------------------------------------------------------------------------------------------------------------------------------

--quantidade de SKUs (FEITO) vermelho (fonte)
--valor faturado

SELECT id, sum(valor), Data  FROM (
SELECT SKU_ID AS id, INS_DATE as Data,
 (SELECT sum(SALE_PRICE) FROM AC_ADMIN.ECAD_ORDER_LINE LINE WHERE STORE_ID = 'NCMP' AND line.SKU_ID= sku.SKU_ID) valor
FROM AC_ADMIN.ECAD_SKU sku WHERE STORE_ID = 'NCMP' AND INS_DATE between '01/01/2018' and current_date
) WHERE valor >= 0
group by id, Data
order by Data ASC;


select * from AC_ADMIN.ECAD_ORDER_LINE;


--------------------------------------------------------------------------------------------------------------------------------------------
-- -- pedidos com tracking


select count(pedido.order_id), pedido.INS_DATE from AC_ADMIN.ECAD_ORDER pedido
where  pedido.INS_DATE between '01/07/2018' and '31/07/2018' and
       pedido.XML_UPDATED like '%issuerDocumentNr%' and
       pedido.STORE_ID = 'NCMP'
      group by pedido.INS_DATE;

;

-- ----------------------------------------------------------------------------------------------------------------------------------------------

-- pedidos aprovados (FEITO)

select sum(order_id) from (
select ss.sku_id, ss.sku_id_origin, ss.store_qualifier_id, o.order_id, o.purchase_date, O.ORDER_AMOUNT from ac_admin.ecia_imported_product p
inner join ac_admin.ecia_importer_info i
on p.id_importer_info = i.id_importer_info
inner join ac_admin.ecma_sku_mp_related_sku_seller ss
on ss.sku_id = p.sku_id and ss.store_qualifier_id = i.shop_code and ss.product_id = p.product_id and trim(p.skuidorigin) = ss.sku_id_origin
inner join ac_admin.ecad_order_line ol
on ol.sku_id = ss.sku_id and ol.store_id = ss.store_id and ol.sku_id_origin = SS.SKU_ID_ORIGIN
inner join ac_admin.ecad_delivery d
on d.delivery_id = ol.delivery_id and d.store_id = ol.store_id
inner join ac_admin.ecad_order o
on o.order_id = d.order_id and o.store_id = d.store_id and o.shop_code = ss.store_qualifier_id
where
	ss.INS_DATE > (TO_DATE('2018-06-05 23:00:00', 'YYYY-MM-DD HH24:MI:SS')) and
	p.id_product_status = 1 and
	i.id_status = 26 and
	i.DT_CREATED > (TO_DATE('2018-06-05 23:00:00', 'YYYY-MM-DD HH24:MI:SS')) and
	o.purchase_date > (TO_DATE('2018-06-05 23:00:00', 'YYYY-MM-DD HH24:MI:SS')) and order_state_tp_id <> 'CAN');

-- -----------------------------------------------------------------------------------------------------------------------------------------------
skus cadastrados (FEITO)

select count (DISTINCT (sku.sku_id)), EXTRACT(month from sku.ins_date) from AC_ADMIN.ECAD_SKU sku
WHERE sku.STORE_ID = 'NCMP'
AND ins_date >= to_date('01-JAN-2018','DD-MON-YY')
GROUP BY EXTRACT(month from sku.INS_DATE) order by EXTRACT(month from sku.INS_DATE);

-- ----------------------------------------------------------------------------------------------------------------------------

--BI - 81  AZUL Claro 221 235 247

-- ---------------------------------------------------------------------------------------------------------------------------
-- BI 83 VERDE comum 102 (FEITO EM PLANILHA)


--BI - 84 - verde claro 198 224 180 (FEITO EM PLANILHA)


-- BI 86 - bege 255 230 153 (FEITO EM PLANILHA)

-- BI 87 - roxo 112 48 160 (FEITO EM PLANILHA) recontato

-- BI 90  - roma
