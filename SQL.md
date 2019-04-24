# SQL

## 1.mysql

## 2.orale

### 1.mybatis中foreach的用法

```html
<select id="selectTestForEach" parameterType="News" resultMap="NewsResultMapper">

  select * from t_news n where 

  <foreach collection="listTag" index="index" item="tag" open="("

​    separator="," close=")">

   \#{tag} in n.tags

  </foreach>
```

https://www.cnblogs.com/dflmg/p/6398033.html

 看。 foreach 生成的效果是集合 转化为下面的

 select * from t_news n where ( ? in n.tags , ? in n.tags )

 查询集合中哪个在n.tags中

### 2.mybatis    用方法in时数据个数大于1000

<select id="selectAssemableNameListByCodeList"  resultMap="assembleResult">
    select  CODE,LONGNAME from T31_HIP_PORTFOLIO
    <where>
        CODE in
        <foreach collection="list" item="item" open="(" close=")" index="index" separator=",">
            <if test="(index % 999) == 998"> NULL ) OR CODE IN (</if>#{item}
        </foreach>
    </where>
</select>

select  CODE,LONGNAME from T31_HIP_PORTFOLIO where code in item(item是被分割的字段)

### 3.distinct返回不重复的值

### 4.查询日期

```
SELECT * FROM cdwdg.t31_edm_trade_day tt WHERE tt.DAY_DT =to_date('2010926','yyyyMMdd')

date'2015-09-26';

to_date('2010926','yyyyMMdd')
```

### 5.查询指定日期的前几天

1.mybatis中foreach的用法

```
select  *
     from cdwdg.t31_edm_trade_day tt
    where tt.sseszse_trade_ind=1 and TT.DAY_DT < DATE'2019-01-01'and ROWNUM <=(7+1) order by TT.DAY_DT desc;
```

```
整体的例子
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.clamc.climb.mapper.hs.EdmTradeDayMapper">
    <resultMap id="BaseResultMap" type="com.clamc.commondto.rtvaluations.EdmTradeDayInfoPO">
        <result column="DAY_DT" jdbcType="DATE" property="dayDt"/>
        <result column="NATURAL_DAY_IND" jdbcType="VARCHAR" property="naturalDayInd"/>
        <result column="SSESZSE_TRADE_IND" jdbcType="VARCHAR" property="sseszseTradeInd"/>
        <result column="INTERBANK_TRADE_IND" jdbcType="VARCHAR" property="interbankTradeInd"/>
        <result column="CFFE_TRADE_IND" jdbcType="VARCHAR" property="cffeTradeInd"/>
        <result column="HKEX_TRADE_IND" jdbcType="VARCHAR" property="hkexTradeInd"/>
        <result column="CW_TRADE_IND" jdbcType="VARCHAR" property="cwTradeInd"/>
        <result column="NASDAQ_TRADE_IND" jdbcType="VARCHAR" property="nasdaqTradeInd"/>
        <result column="NYSE_TRADE_IND" jdbcType="VARCHAR" property="nyseTradeInd"/>
        <result column="HGT_TRADE_IND" jdbcType="VARCHAR" property="hgtTradeInd"/>
    </resultMap>
    <sql id="Base_Column_List">
        DAY_DT,NATURAL_DAY_IND,SSESZSE_TRADE_IND,INTERBANK_TRADE_IND,INTERBANK_TRADE_IND,
        CFFE_TRADE_IND,HKEX_TRADE_IND,CW_TRADE_IND,NASDAQ_TRADE_IND,NYSE_TRADE_IND,HGT_TRADE_IND
    </sql>
    <!--查询日期区间交易日天数,备付金调用-->

<select id="SelectTradeDay" resultMap="BaseResultMap"  parameterType="map">
select  <include refid="Base_Column_List"/>
    select * from cdwdg.t31_edm_trade_day tt
    where tt.sseszse_trade_ind=1 and TT.DAY_DT &lt; DATE'#{bizDate}'and ROWNUM &lt;=(#{days}+1) order by TT.DAY_DT desc;
</select>

</mapper>
```

```
<select id="selectBeforeTradeDay" resultMap="BaseResultMap"  parameterType="map">
select  <include refid="Base_Column_List"/>
     from cdwdg.t31_edm_trade_day tt
    where tt.sseszse_trade_ind=1 and TT.DAY_DT &lt; to_date('bizDate','yyyyMMdd') and ROWNUM &lt;=(#{days}+1) order by TT.DAY_DT desc;
</select>
```

