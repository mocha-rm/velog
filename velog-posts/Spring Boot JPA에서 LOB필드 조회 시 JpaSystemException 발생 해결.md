<h2 id="문제-발생">문제 발생</h2>
<p>Spring Boot + JPA 환경에서 상품 단건 조회 API를 호출했을 때 다음과 같은 예외가 발생했다.</p>
<pre><code>org.springframework.orm.jpa.JpaSystemException: Unable to access lob stream  
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:341)
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:241)</code></pre><ul>
<li>문제가 된 코드</li>
</ul>
<pre><code class="language-java">@Override
public ProductResponseDto findProduct(Long productId) {
    Product product = productRepository.findById(productId)
            .orElseThrow(() -&gt; new GlobalException(ProductErrorCode.PRODUCT_NOT_FOUND));
    return ProductResponseDto.toDto(product);
}</code></pre>
<p>Product 엔티티는 @Lob 타입의 description 필드를 포함하고 있음
ProductPhoto 엔티티와 @OneToMany 관계를 맺고 있으며 이를 조회할 때 @EntityGraph를 사용했음</p>
<h2 id="원인-추론">원인 추론</h2>
<ol>
<li>Lazy 로딩으로 인한 트랜잭션 종료 문제</li>
</ol>
<ul>
<li>@Lob 필드는 기본적으로 Lazy 로딩이다.</li>
<li>@EntityGraph(attributePaths = {&quot;productPhotos.photo&quot;})를 설정했지만, @Lob 필드는 즉시 로딩되지 않음.</li>
<li>트랜잭션 종료 후 description 필드에 접근하려다 보니 JpaSystemException이 발생.</li>
</ul>
<ol start="2">
<li>해당 필드가 LOB 타입이라 직접적인 스트림 접근이 필요함.</li>
</ol>
<ul>
<li>Hibernate는 LOB 필드를 Lazy 로딩할 때 스트림을 통해 접근하는데, 트랜잭션이 끝나면 세션이 닫혀서 접근 불가.</li>
</ul>
<h2 id="해결-방안">해결 방안</h2>
<ol>
<li>@EntityGraph를 사용해 연관 데이터를 즉시 로딩
기존에는 productPhotos.photo만 즉시 로딩되었지만, description 필드는 포함되지 않았다.
따라서 @EntityGraph를 유지하면서 트랜잭션 범위를 확장해야 했다.</li>
</ol>
<pre><code class="language-java">@EntityGraph(attributePaths = {&quot;productPhotos.photo&quot;})
Optional&lt;Product&gt; findProductWithPhotoById(Long id);</code></pre>
<ol start="2">
<li>@Transactional(readOnly = true)로 트랜잭션 범위 확장
트랜잭션이 유지되면 Lazy 로딩 필드(description)에도 안전하게 접근할 수 있다.</li>
</ol>
<pre><code class="language-java">@Override
@Transactional(readOnly = true) // 트랜잭션 유지
public ProductResponseDto findProduct(Long productId) {
    Product product = productRepository.findProductWithPhotoById(productId)
            .orElseThrow(() -&gt; new GlobalException(ProductErrorCode.PRODUCT_NOT_FOUND));
    return ProductResponseDto.toDto(product);
}</code></pre>
<h2 id="결과-확인">결과 확인</h2>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/0334e528-c0d6-4f48-86bf-fabef9fa44e4/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/jelog_131/post/3da5c6ba-cdf7-492b-98dd-4d7c57f5a43d/image.png" /></p>
<ul>
<li>Product를 조회할 때 productPhotos.photo와 함께 description 필드도 정상적으로 로딩됨.</li>
<li>JpaSystemException이 발생하지 않고, 트랜잭션이 유지된 상태에서 @Lob 필드에 접근 가능.</li>
<li>불필요한 추가 쿼리 없이 한 번의 쿼리로 데이터를 조회할 수 있어 성능도 개선됨.</li>
</ul>
<h4 id="key-point">Key Point</h4>
<ul>
<li>@Lob 필드는 기본적으로 Lazy 로딩되므로, 조회 시 트랜잭션을 유지하는 것이 중요 ! </li>
<li>@EntityGraph를 사용하면 특정 필드를 즉시 로딩할 수 있지만, @Lob 필드는 별도로 고려해야 함 !</li>
<li>트랜잭션이 종료되기 전에 데이터를 모두 조회해야 LazyInitializationException을 방지할 수 있음 !</li>
</ul>