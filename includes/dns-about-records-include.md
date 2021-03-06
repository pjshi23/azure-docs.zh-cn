## <a name="about-records"></a>关于记录

每个 DNS 记录都有一个名称和类型。 这些记录根据其所包含的数据分为各种类型。 最常见的类型为“A”记录，这种记录将名称映射到 IPv4 地址。 另一种类型是“MX”记录，这种记录将名称映射到邮件服务器。

Azure DNS 支持所有常见 DNS 记录类型，包括 A、AAAA、CNAME、MX、NS、PTR、SOA、SRV 和 TXT。 请注意：

* 每个区域的 SOA 记录集都是自动创建的，不能单独创建。
* 应使用 TXT 记录类型创建 SPF 记录。 有关详细信息，请参阅 [本页](http://tools.ietf.org/html/rfc7208#section-3.1)。

在 Azure DNS 中，记录使用相对名称指定。 完全限定域名 (FQDN) 包括区域名称，而相对域名则不包括。 例如，区域“contoso.com”中的相对记录名称“www”会提供完全限定记录名称 www.contoso.com。

## <a name="about-record-sets"></a>关于记录集

有时，需要创建具有给定名称和类型的多个 DNS 记录。 例如，假设在两个不同的 IP 地址上托管“www.contoso.com”网站。 该网站需要两个不同的 A 记录，每个 IP 地址一个。 这就是记录集：

    www.contoso.com.        3600    IN    A    134.170.185.46
    www.contoso.com.        3600    IN    A    134.170.188.221

Azure DNS 使用记录集来管理 DNS 记录。 记录集是某个区域中具有相同名称、相同类型的 DNS 记录的集合。 大多数记录集都包含单个记录，但像这样的包含多个记录的记录集也不少见。

SOA 和 CNAME 记录集例外。 DNS 标准不允许这些记录集类型包含具有相同名称的多个记录。

生存时间（或 TTL）指定客户端在重新查询之前缓存每个记录的时长。 在本例中，TTL 为 3600 秒或 1 小时。 TTL 针对记录集而非每个记录指定，因此同一个值适用于该记录集中的所有记录。

#### <a name="wildcard-record-sets"></a>通配符记录集

Azure DNS 支持 [通配符记录](https://en.wikipedia.org/wiki/Wildcard_DNS_record)。 具有匹配名称的任何查询都会返回这些记录集（除非存在与非通配符记录集更接近的匹配项）。 除 NS 和 SOA 外的所有记录类型都支持通配符记录集。

若要创建通配符记录集，请使用记录集名称“\*”。 或者，使用具有“\*”标签的名称，例如“\*.foo”。

#### <a name="cname-record-sets"></a>CNAME 记录集

CNAME 记录集不能与其他具有相同名称的记录集共存。 例如，不能同时创建具有相对名称“www”的 CNAME 记录集和具有相对名称“www”的 A 记录。 由于区域顶点（名称 = ‘@’)）始终包含创建区域时创建的 NS 和 SOA 记录集，因此不能在区域顶点创建 CNAME 记录集。 这些约束起源于 DNS 标准，并非 Azure DNS 的限制。


<!--HONumber=Nov16_HO2-->


