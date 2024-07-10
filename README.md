# ENRICH_ELASTICSEARCH


## crear indice que nos dara los datos de enriquecimiento ##


      PUT /etl_base
      {
        "mappings": {
          "properties": {
            "olt": {
              "type": "keyword"
            },
            "ciudad": {
              "type": "text"
            },
            "provincia": {
              "type": "text"
            }
          }
        }
      }
      
## crear indice al que vamos a enriquecer

      PUT /crm_info
      {
        "mappings": {
          "properties": {
            "olt_crm": {
              "type": "keyword"
            }
          }
        }
      }

## insertar informacion a etl_base para hacer el escenario


    POST /etl_base/_bulk
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a2", "ciudad": "Guayaquil", "provincia": "Guayas" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a3", "ciudad": "Cuenca", "provincia": "Azuay" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a4", "ciudad": "Santo Domingo", "provincia": "Santo Domingo de los Tsáchilas" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a5", "ciudad": "Machala", "provincia": "El Oro" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a6", "ciudad": "Ambato", "provincia": "Tungurahua" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a7", "ciudad": "Manta", "provincia": "Manabí" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a8", "ciudad": "Portoviejo", "provincia": "Manabí" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a9", "ciudad": "Loja", "provincia": "Loja" }
    { "index": { "_index": "etl_base" } }
    { "olt": "olt_a10", "ciudad": "Ibarra", "provincia": "Imbabura" }



# COMENZAMOS EL PROCESO ENRICH #
##--------creamos la politica enrich----------####
##---llamaremos a esta politica "enrich_base_etl"---##


    PUT /_enrich/policy/enrich_base_etl
    {
      "match": {
        "indices": ["etl_base"],
        "match_field": "olt",
        "enrich_fields": ["ciudad", "provincia"]
      }
    }
    
#---------ejecutar la politica enrich despues de haberla ejecutado----#####

    POST /_enrich/policy/enrich_base_etl/_execute


#--------crear el ingest papeline que llama al enrich creado------##
###-----llamaremos esta ingespepeline: "pipeline_enrich_crm_info"-----###
###-----podemos hacer mas cosas aqui con los datos si se desea-----#########

    PUT /_ingest/pipeline/pipeline_enrich_crm_info
    {
      "processors": [
        {
          "enrich": {
            "policy_name": "enrich_base_etl",
            "field": "olt_crm",
            "target_field": "enrich.",
            "max_matches": "1"
            
          }
        }
      ]
    }
####-----subimos el pepeline al crm_info para que se enriquezca automaticamente------######

    PUT crm_info/_settings
    {
      "index": {
        "default_pipeline": "pipeline_enrich_crm_info"
      }
    }

####-----refrescamos el indice para que tome esa nueva configuracion---####

    POST /crm_info/_refresh
####---cargamos un documento con solo un dato, debe enriquecerse solo desde la base etl_base--------####

    POST /crm_info/_doc/1
    { "olt_crm": "olt_a2" }
    
###veamos la info que nos dio ya enriquecida

    GET /crm_info/_doc/1
    
###respuesta:

            {
              "_index" : "crm_info",
              "_type" : "_doc",
              "_id" : "1",
              "_version" : 5,
              "_seq_no" : 19,
              "_primary_term" : 1,
              "found" : true,
              "_source" : {
                "olt_crm" : "olt_a2",
                "enrich" : {
                  "olt" : "olt_a2",
                  "ciudad" : "Guayaquil",
                  "provincia" : "Guayas"
                }
              }
            }


################################################

    GET /_enrich/policy
    GET /crm_info/_settings
    GET /etl_base/_mapping
    POST /crm_info/_refresh
