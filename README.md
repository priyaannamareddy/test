ABAP Package: ZTEMP_P
-----------------------------
Database Table: ZORG_PLANNING
@EndUserText.label : 'Orgransation Plan data'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #RESTRICTED
define table zorg_planning {
  key client  : abap.clnt not null;
  retailerid  : abap.char(30);
  sku         : abap.char(10);
  description : abap.char(30);
  theme       : abap.char(30);

}

ABAP Class: ZCL_ORG_PLANDATA_UP
----------------------------------------
CLASS ZCL_org_plandata_up DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.
  INTERFACES if_oo_adt_classrun.
  PROTECTED SECTION.
  PRIVATE SECTION.
ENDCLASS.



CLASS ZCL_org_plandata_up IMPLEMENTATION.
METHOD if_oo_adt_classrun~main.
 DATA:it_custfields TYPE TABLE OF zorg_planning.

*   fill internal table (itab)
    it_custfields = VALUE #(
        (
        client = sy-mandt retailerid = 'AMZ' SKU = '10295'  Description = 'Porsche 911'
          theme = 'Creator Expert' ) ).

*   Delete the possible entries in the database table - in case it was already filled
    DELETE FROM zorg_planning.
*   insert the new table entries
    INSERT zorg_planning FROM TABLE @it_custfields.

*   check the result
    SELECT * FROM zorg_planning INTO TABLE @it_custfields.
    out->write( 'Data inserted successfully!').
endmethod.
ENDCLASS.

Data Definition (Interface View): ZI_CUST_TABLE
----------------------------------------

@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Interface view'
define root view entity Zi_cust_table
  as select from zorg_planning as Planningdata 
{ key  compcode,
  retailerid,
  sku,          
  description, 
  theme 
}  


Data Definition (Projection View): ZC_PVIEW
----------------------------------------

@EndUserText.label: 'Projection view - Processor'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@UI: {
 headerInfo: { typeName: 'Planningdetails', title: { type: #STANDARD } } }

@Search.searchable: true
  
define root view entity ZP_VIEW  as projection on Zcds_cust_table
{
      @UI.facet: [ { 
                     purpose:         #STANDARD,
                     type:            #IDENTIFICATION_REFERENCE,
                     label:           'Planning Details',
                     position:        10 } ]
     
      @UI: {
          lineItem:       [ { position: 01, importance: #MEDIUM } ],
          identification: [ { position: 01 , label: 'RetailerId'} ] }
        @Search.defaultSearchElement: true
  key compcode              as RetailerId,
 
      @UI: {
          lineItem:       [ { position: 10, importance: #MEDIUM } ],
          identification: [ { position: 10 , label: 'SKU'} ] }
        @Search.defaultSearchElement: true
      sku          as Sku,

        @UI: {
          lineItem:       [ { position: 40, importance: #MEDIUM } ],
          identification: [ { position: 40, label: 'Description' } ] }
          @Search.defaultSearchElement: true
      description         as Description,

      @UI: {
          lineItem:       [ { position: 41, importance: #MEDIUM } ],
          identification: [ { position: 41,label: 'Theme' } ] }
          @Search.defaultSearchElement: true
      theme           as Theme
}



Service Definition: ZS_CUST and Service Binding: ZSERVICE_BUIDINGS View
-------------------------------
@EndUserText.label: 'Service Def.'
define service ZS_CUST {
  expose ZP_VIEW as Planningdetails;
}

Behavior Definition: ZCL_UNQIECLASS
----------------------------------

CLASS zcl_unqieclass DEFINITION
 PUBLIC
ABSTRACT
FINAL
FOR BEHAVIOR OF Zcds_cust_table.
ENDCLASS.



CLASS zcl_unqieclass IMPLEMENTATION.
*Modify DB Table
ENDCLASS.


define behavior for ZCDS_CUST_TABLE alias Travel
persistent table zorg_planning
etag master retailerid
lock master
{
   
    // mandatory feilds that are required to create 
    field( mandatory ) description,retailerid,sku,theme;

// standard operations   
    create;
    update;
    delete;
}


