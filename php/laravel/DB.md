#  select

引用  use Illuminate\Support\Facades\DB;

```php
// 按顺序设置 where条件的参数
$bindPrramArray = [$searchVal["kyoku_no"], $searchVal["koubai_nendo"], $searchVal["koubai_no"], $searchVal["koubai_kind"], $searchVal["baikyaku_no"]];
// make sql 语句
$basisSql =    "SELECT" .
                " * " . // 局署番号
                " FROM (hudousan_kihon as a" .
                " LEFT JOIN zaisan_bunrui AS b ON (a.zaisan_bunrui = b.zaisan_bunrui))" .
                " LEFT JOIN rosen_name AS c ON (a.koutsu_kikan_1 || a.rosen_1 = c.rosen_id)" .
                " LEFT JOIN rosen_name AS d ON (a.koutsu_kikan_2 || a.rosen_2 = d.rosen_id)" .
                " LEFT JOIN rosen_name AS e ON (a.koutsu_kikan_3 || a.rosen_3 = e.rosen_id)" .
                " LEFT JOIN kyokusho_info AS f ON (a.kyoku_no = f.kyoku_no)";

/* 検索条件追加	 */
$basisSql .= " WHERE      a.kyoku_no =  ? " . // 局署番号
    " AND a.koubai_nendo =  ? " . // 公売年度
    " AND a.koubai_no =  ? " . // 公売公告番
    " AND a.koubai_kind =  ? " . // 公売種類
    " AND a.baikyaku_kubun_no =  ? ";   // 売却区分番

// 执行sql 语句，返回一数组
$basisData = DB::select($basisSql, $bindArray);
if (!isset($basisData[0])) {
    //存在しない場合はTOPにリダイレクト
    return redirect('auctionx/public/hp001.php');
}
```

