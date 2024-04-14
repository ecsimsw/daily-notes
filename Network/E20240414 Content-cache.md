cloudfront 기본 ttl : 24h

Cache-control    
no-cache, max-age=0      
no-cache : 원본 확인
cache 있는데 age 지남 -> 사용할 때 원본 서버에 시간, Etag 등으로 비교          
no-store : 캐시를 하지 않음      

Private cache : browser     
Shared cache : Managed(CDN, reverse proxy), Proxy

