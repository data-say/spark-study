## Web Notebook

- 장점
    - 대화형 분석 가능
    - 작성한 코드 저장 가능
    - 언어별 차트 라이브러리 및 노트 자체 차트 기능을 통해 분석 내용 시각화 가능
- 종류
    - Apache Zeppelin
    - Jupyter Notebook
    - RStudio Server

## Apache Zeppelin

- `https://zeppelin.apache.org`
- 하나의 노트에서 여러 클러스터에 접근 가능
- 하나의 노트에서 여러 언어로 Spark 코드 작성 가능
- 하나의 노트에서 Spark 외의 다른 Shell도 실행 가능
- zeppelin 자체 기능으로 시각화 가능
    
    ```scala
    z.show(df)
    ```
    

## Jupyter Notebook

- `https://jupyter.org`
- spark df → pandas df
    
    ```python
    pdf = df.groupBy("Year", "Month").agg(func.count("*").alias("count")).toPandas()
    ```
    
- python 에서 R의 ggplot2를 사용하여 시각화하는 방법
    
    ```python
    # !pip install plotnine
    %matplotlib inline
    import plotnine as p9
    
    p9.ggplot(data=pdf, mapping=p9.aes(x="Year", y="count", fill="Month")) +
    					p9.geom_bar(stat='identity') +
    					p9.coord_flip()
    ```
    

## RStudio Server

- `https://www.rstudio.com`
- R 시각화
    
    ```r
    ggplot(data=r_df, mapping=aes(x=Year, y=count, fill=Month)) + 
    		geom_bar(stat='identity') +
    		coord_flip()
    ```
