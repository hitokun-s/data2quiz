leaflet-providorで、地図のスタイル変更
[https://leaflet-extras.github.io/leaflet-providers/preview/](https://leaflet-extras.github.io/leaflet-providers/preview/)

マーカーの基本コード
[http://www.slideshare.net/takahi/leafletjs-44404366](http://www.slideshare.net/takahi/leafletjs-44404366)

API例
http://nominatim.openstreetmap.org/search?country=japan&format=json&viewbox=139.5744323730469,35.74846274418971,139.9040222167969,35.60902228231369

R,N,Wについて
http://www.openstreetmap.org/relation/3599783

Overpass APIについて
http://qiita.com/taku/items/f3c6ed375a2461551d7c

Overpass API例
http://wiki.openstreetmap.org/wiki/JA:Overpass_API/%E8%A8%80%E8%AA%9E%E3%82%AC%E3%82%A4%E3%83%89

「都市」のnodeのタグ
http://wiki.openstreetmap.org/wiki/Tag:place%3Dcity

主要都市と座標データ
http://simplemaps.com/resources/world-cities-data

その他公開データ
http://standardization.at.webry.info/201101/article_6.html

overpass api練習サイト
http://overpass-turbo.eu/

上田周辺の町を問い合わせる例
URL: www.overpass-api.de/api/interpreter

POSTデータ：

    [out:json];
    node
       [place=town]
       (36.3073788467898,137.97454833984378,36.58355488335725,138.63372802734378);
    out;

#trouble shooting

```$.post( "https://overpass-api```を```$.post( "https://www.overpass-api```にした途端、CORS問題？でエラーや警告になる。

- chrome => securityタブにエラーが出て、アドレスバーのhttpsが×印になる
- IE11 => access deniedになる

#leafletのURLでhttpsのものにするには？
https://github.com/Leaflet/Leaflet/issues/3186

#行政区界を取得する
http://wiki.openstreetmap.org/wiki/JA:%E5%A2%83%E7%95%8C%E7%B7%9A

githubのマップはmapbox×osmを使っている

よくみる、mapboxのlight/darkスタイルはこれ。
https://www.mapbox.com/maps/light-dark/
これは「ベースマップ」の一種らしい。
http://dev.classmethod.jp/etc/getting-started-mapbox/

#ランダムに地点を選ぶ方法案
1. 国か何かを絞ったうえで、osmのIDを使えばいいのでは？海の上にはたぶんobjectはないはず・・・
でもidが連続している保証はないので、この作戦は無理。
2. 国を指定した上で、「国」に次ぐ行政レベル（state?）を指定して一覧をとればよいのでは？そこからランダムに選ぶ。

#overpass-APIで境界を取得する例

data=[out:json];rel[boundary=administrative][admin_level=6](35.48695192624445,139.09515380859378,35.62660519422425,139.42474365234378);out;

肝心なのは、クイズ対象都市の境界だけ表示すること！

これも参考：
http://gis.stackexchange.com/questions/76869/how-to-get-wkt-or-geojson-by-country-or-city-name-with-overpass-api
「area」で直接問い合わせているのか？？？

日本の全都市を取得する。

data=[out:json];(node[place=city]["is_in:country_code"="JP"];);out;

お、人口もクエリにできるっぽい。
http://wiki.openstreetmap.org/wiki/Tag:place%3Dcity
これは、クイズのヒントにできるな。。。

placeに設定されるコード一覧
http://wiki.openstreetmap.org/wiki/Map_Features

各国のjsonデータ
https://mledoze.github.io/countries/

日本の全市区町村のgeoJsonとtopoJson
https://github.com/niiyz/JapanCityGeoJson

overpass-APIで、ある領域内の河川wayは取得できる。

way
  [waterway=river]
  ({{bbox}});
/*added by auto repair*/
(._;>;);
/*end of auto repair*/
out;

しかし、長さや規模でフィルタリングができない！
なので、主要河川のクイズには使えない。

また同様に、都市を人口で絞ることも難しい。
正規表現？で頑張る例：

[timeout:25];
(
  node["place"="city"]

  ["population"~"^4.....$|^5.....$|^3.....$"] /* between 100000 - 399999*/
  ({{bbox}});
);
out body;

代わりに、dbpediaを使えないだろうか？
参考：http://qiita.com/pika_shi/items/eb56fc205e2d670062ae

なんかdbpediaを検索していけるサービス：
例：
http://dbpedia.org/describe/?url=http%3A%2F%2Fdbpedia.org%2Fresource%2FPanchagangavalli_River&sid=3639
「body of water」とかのtagが気になるが、wikipediaページ
https://en.wikipedia.org/wiki/Panchagangavalli_River
にはそんなデータはない。

でもアマゾン川なら
https://en.wikipedia.org/wiki/Amazon_River
- length
- basin（流域面積）
- source（水源？）
- mouth（河口？）
が載っている！

神資料
https://www.w3.org/2009/Talks/0615-qbe/
http://wp.lodosaka.jp/tool/sparqlquery/
https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries/examples

（例）「日本百」名山の標高順

http://ja.dbpedia.org/sparql
で、
------------------------------------------------------
select *
where
{
  ?uri dbpedia-owl:wikiPageWikiLink category-ja:日本百名山.
  ?uri rdf:type schema:Mountain.
  ?uri foaf:name ?name.
  ?uri foaf:depiction ?image.
  ?uri prop-ja:標高 ?altitude.
}
order by ?altitude
------------------------------------------------------

世界の川
http://dbpedia.org/sparql
で、
PREFIX dbpedia-owl: <http://dbpedia.org/ontology/>
SELECT *
WHERE {
   ?s a dbpedia-owl:River .
}


wikidata query というページだとグラフまで出せる！！
https://query.wikidata.org/

https://query.wikidata.org/#%23defaultView%3ALineChart%0ASELECT%20%3Fcountry%20%20%3Fyear%20%3Fpopulation%20%3FcountryLabel%20WHERE%20%7B%0A%20%20%7B%0A%20%20%20%20SELECT%20%3Fcountry%20%3Fyear%20%28AVG%28%3Fpopulation%29%20AS%20%3Fpopulation%29%20WHERE%20%7B%0A%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20SELECT%20%3Fcountry%20%28str%28YEAR%28%3Fdate%29%29%20AS%20%3Fyear%29%20%3Fpopulation%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%3Fcountry%20wdt%3AP47%20wd%3AQ183.%0A%20%20%20%20%20%20%20%20%20%20%3Fcountry%20p%3AP1082%20%3FpopulationStatement.%0A%20%20%20%20%20%20%20%20%20%20%3FpopulationStatement%20ps%3AP1082%20%3Fpopulation.%0A%20%20%20%20%20%20%20%20%20%20%3FpopulationStatement%20pq%3AP585%20%3Fdate.%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20GROUP%20BY%20%3Fcountry%20%3Fyear%0A%20%20%7D%0A%20%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22en%22.%20%7D%0A%7D

お、川があった！
https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries/examples#Rivers
山もある
https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries/examples#Mountains

「wd」「wdt」などの意味はこちら：
https://www.mediawiki.org/wiki/Wikibase/Indexing/RDF_Dump_Format#Prefixes_used

apiでjsonで取得する例

https://www.mediawiki.org/wiki/Wikidata_query_service/User_Manual#SPARQL_endpoint
に説明がある。
（例）
https://query.wikidata.org/sparql?query=SELECT ?subj ?label ?coord ?elev  WHERE { 	?subj wdt:P2044 ?elev	filter(?elev > 8000) . 	?subj wdt:P625 ?coord . 	SERVICE wikibase:label { bd:serviceParam wikibase:language "en,zh" . ?subj rdfs:label ?label }  }&format=json

※queryパラメータの値はたぶんURLエンコード必要。

【mountainやriverの他にどんな地形が登録されているのか？】
mountainの
http://dbpedia.org/ontology/Mountain
をみると、親クラスは、
http://dbpedia.org/ontology/NaturalPlace

- Archipelago（諸島）
- Beach
- BodyOfWater
- Cave
- Crater
- Desert
- Glacier（氷河）
- HotSpring
- Mountain
- MountainPass
- MountainRange
- Valley
- Volcano

ちなみに、
http://dbpedia.org/ontology/Mountain
の「disjointWith：Person」がすごく気になる。

川は、
http://dbpedia.org/ontology/River
http://dbpedia.org/ontology/Stream
- River
- Creek（小川）
- Canal
http://dbpedia.org/ontology/BodyOfWater
- Bay
- Lake
- Ocean
- Sea
- Stream

「半島」「湾」「岬」「海峡」「高原」「盆地」「台地」などは？=> ontologyにないっぽい？

でもwikipediaカテゴリーには「半島」はある。
https://en.wikipedia.org/wiki/Category:Peninsulas

オントロジー一覧！
http://mappings.dbpedia.org/server/ontology/classes/
やっぱりpeninsulaはない！

http://dbpedia.org/fct/
からキーワード検索してみる。
http://dbpedia.org/fct/facet.vsp?cmd=text&sid=3646
すると、各「半島」データのrdf:typeに、「yago:Peninsula109388848」というyagoラベルがついている。
http://dbpedia.org/describe/?url=http%3A%2F%2Fdbpedia.org%2Fclass%2Fyago%2FPeninsula109388848&distinct=1&p=1&sid=3646&lp=12&op=-1&next=&gp=1

これでqueryできれば良い。

http://km.aifb.kit.edu/projects/spartiqulator/v5/examples.htm
をみると、

    PREFIX yago: <http://dbpedia.org/class/yago/>

とした上で、

    ?city rdf:type yago:StatesOfGermany .

のように書けるらしい。

こんな風に書いてみたが、

PREFIX yago: <http://dbpedia.org/class/yago/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?uri ?string
WHERE
{
	?uri rdf:type yago:WikicatPeninsulasOfItaly .
	OPTIONAL {?uri rdfs:label ?string . FILTER (lang(?string) = 'en') }
}
limit 5

うまくいかない。。。
と思ったら、
http://km.aifb.kit.edu/projects/spartiqulator/v5/examples.htm
のサンプルはどれも結果を拾えない。

参考これ？
https://github.com/nicolas-fricke/semmul2014/wiki/Results:-DBpedia-interlinking,-Query-improvement,-Wikidata

wikidataサイトで検索すると「peninsula」がある
https://www.wikidata.org/wiki/Q34763

具体例はこういうの。
https://www.wikidata.org/wiki/Q2603732
「coordinate location」がある。領域範囲はわかるのかな？
あとlocationだけだと「有名な半島」だけ絞り込むことができないので、クイズにしにくい。
バートン半島：
https://en.wikipedia.org/wiki/Barton_Peninsula
でも領域が表示されない。
ペロポネソス半島：
https://www.wikidata.org/wiki/Q78967
でもバートン半島よりはプロパティが多いな。。。

例えばRiverをdbpediaから取得したとしても、その川の経路（geoJson？）はoverpass-APIから取得することになる？？

「手取川」ルートを取得する例：
http://overpass-turbo.eu/
で試せる。

[out:json][timeout:25];
// gather results
(
  // query part for: “waterway=river”
  way["waterway"="river"]["name"="手取川"];
);
// print results
out body;
>;
out skel qt;

レスポンスはこういう感じ：
----------------------------------------------------
[
    {
        "type": "way",
        "id": 24332000,
        "nodes":[
            279948639,
            444675842,
            ...
-------------------------------------
まさか、このnodeを１つずつ問い合わせて、経緯を取得する必要があるのか？？
同じ質問：
http://stackoverflow.com/questions/25297077/how-can-i-get-a-list-of-lat-lon-pairs-for-a-way-in-a-single-call-to-the-osm-a
個別のwayを問い合わせるだけみたい？
いや違う。

[out:json];way(137281421);out;

とやっても、相変わらずnodeIDしかない。
ちがう、こうだ！

[out:json];way(137281421);>;out;

最終的にはこう。
------------------------------------
[out:json];
(
  way["waterway"="river"]["name"="手取川"];>;
);
out body;
>;
out skel qt;
------------------------------------

skelについて：
http://wiki.openstreetmap.org/wiki/Overpass_API/Language_Guide#Concise_.28skeleton.2C_skel.29

qtについて：
http://wiki.openstreetmap.org/wiki/Overpass_API/Language_Guide#Order_attribute_for_faster_results_.28quadtile.2C_qt.29
「allow elements to be ordered by their location 」
これはwayの可視化のためには絶対に必要なのでは？？？

【まとめ】
dbpediaで取得した結果の川の名称を、overpass-APIクエリのnameに入れればいい。

【icon画像を探す】
http://www.flaticon.com/search?word=river
※ページのフッター等に著者表示をすればライセンスOKらしい。




















































