# 环境
python3.7（python3应该都没问题）

# 评估
输入文件格式：
1. 文件以.sql结尾
2. 文件每行的格式："qid\tsql_query\tdb_id",其中predcit文件db_id是可选字段，gold文件db_id是必选字段
3. 评估指标：exact matching score

# 数据集格式
## SQL json
### NL2SQL

    "sql":{ 
        "sel": [col_id, ..], # SQL选择的列 
        "agg": [agg_id, ..], # 选择的列相应的聚合函数, '0'代表无
        "cond_conn_op": conn_id, # 条件之间的关系
        "conds": [
            [col_id, col_op, value], # 条件列, 条件类
            ...
        ]
    }

    "sql":{ # 真实SQL
        "sel": [7], # SQL选择的列 
        "agg": [0], # 选择的列相应的聚合函数, '0'代表无
        "cond_conn_op": 0, # 条件之间的关系
        "conds": [
            [1,2,"世茂茂悦府"], # 条件列, 条件类型, 条件值，col_1 == "世茂茂悦府"
            [6,0,1]
        ]
    }
其中：

    op_sql_dict = {0:">", 1:"<", 2:"==", 3:"!="}
    agg_sql_dict = {0:"", 1:"AVG", 2:"MAX", 3:"MIN", 4:"COUNT", 5:"SUM"}
    conn_sql_dict = {0:"", 1:"and", 2:"or"}

### DuSQL & CSpider

    val: number(float)/string(str)/sql(dict)
    col_unit: (agg_id, col_id, isDistinct(bool))
    val_unit: (unit_op, col_unit1, col_unit2)
    table_unit: (table_type, col_unit/sql)
    cond_unit: (agg_id, cond_op, val_unit, val1, val2)
    condition: [cond_unit1, 'and'/'or', cond_unit2, ...]
    sql {
       'select': [(agg_id, val_unit), (agg_id, val_unit), ...]
       'from': {'table_units': [table_unit1, table_unit2, ...], 'conds': condition}
       'where': condition
       'groupBy': [col_unit1, col_unit2, ...]
       'orderBy': ('asc'/'desc', [(agg_id, val_unit), ...])
       'having': condition
       'limit': None/number(int)
       'intersect': None/sql
       'except': None/sql
       'union': None/sql
    }
其中：

    COND_OPS = ('not_in', 'between', '==', '>', '<', '>=', '<=', '!=', 'in', 'like')
    UNIT_OPS = ('none', '-', '+', "*", '/')
    AGG_OPS = ('none', 'max', 'min', 'count', 'sum', 'avg')
    TABLE_TYPE = {
    'sql': "sql",
    'table_unit': "table_unit",
    }

    LOGIC_AND_OR = ('and', 'or')
    SQL_OPS = ('intersect', 'union', 'except')
    ORDER_OPS = ('desc', 'asc')

## db schema
    
    {
    "column_names":[[table_id, column_name]...]
    "table_names":[table_name ...] (NL2SQL中来自name)
    "column_names_original":[[table_id, column_name_original]...]
    "table_names_original":[table_name_original ...] (NL2SQL中来自name)
    "column_types":["text/time/number/.." ...]
    "db_id":db_id (NL2SQL中来自id)
    "foreign_keys":[[col_id, col_id]...]
    "primary_keys":[prime_key, ...]
    }
    
   
## spider db schema
    
    {
    "column_names":[[table_id, column_name]...]
    "table_names":[table_name ...]
    "column_names_original":[[table_id, column_name_original]...]
    "table_names_original":[table_name_original ...]
    "column_types":["text/time/number/.." ...]
    "db_id":db_id
    "foreign_keys":[[col_id, col_id]...]
    "primary_keys":[prime_key, ...]
    }
    

## db content

    "db_id":db_id (NL2SQL中来自table_id)
    "tables": {
        table_name:{
            "cell":[row1, row2, ...]
            "header":[col1, col2, ...]
            "table_name":table_name (NL2SQL中来自name)
            "types":["text/time/number/.." ...]
        }
    }

    CSpider: icfp_1 chinook_1 flight_4 world_1 voter_1 small_bank_1 twitter_1 company_1 epinions_1等database读取content数据出错，部分无content数据

## spider db content

    {
    "db_id":db_id (NL2SQL中来自table_id)
    "tables": {
        table_name:{
            "cell":[row1, row2, ...]
            "header":[col1, col2, ...]
            "table_name":table_name
            "types":["text/time/number/.." ...]
                    }
               }
               
    }

## train/dev

    "question_id":qid
    "db_id":db_id
    "question":question
    "sql":sql_json
    "query":sql(select ? from ? where ?)

## test:

    "question_id":qid
    "db_id":db_id
    "question":question
    "sql":''
    "query":''

# 使用

## 命令行

    python text2sql_eval.py \
        --g 'data/DuSQL/test_gold.sql' \ # gold文件
        --p 'test_DuSQL.sql' \ # predict文件
        --t 'data/DuSQL/db_schema.json' \ # schema文件
        --d 'DuSQL' \ # 选择dataset（DuSQL、NL2SQL、CSPider可选）

## 接口

    from text2sql_evaluation import evaluate
    score, score_novalue = evaluate('table.json', 'gold.sql', 'pred.sql',dataset='DuSQL')
其中：

    score["all"] = {"exact": exact num, "count": test examples num, "acc": accuracy}
    score_novalue["all"] = {"exact": exact num, "count": test examples num, "acc": accuracy}
    
## 输出
    with value:
    {"exact": exact num, "count": test examples num, "acc": accuracy}
    without value:
    {"exact": exact num, "count": test examples num, "acc": accuracy}

    
