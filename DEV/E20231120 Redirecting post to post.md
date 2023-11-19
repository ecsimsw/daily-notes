## E20231120 Redirecting post to post.md

Picup 에서 토큰이 무효한 경우 리프레시 토큰을 확인해서 재발급해주고 리다이렉트를 요구한다.       
그때 301, 302로 응답했다가 Post 요청에서 Get 으로 리다이렉트 되는 현상을 뒤늦게 확인했다.    

리다이렉팅 코드 307, 308로 응답하면 요청 메서드를 살려서 리다이렉트가 가능하다.     

## Different action between Http status

For use cases like bank payments, we might need to redirect an HTTP POST request. Depending on the HTTP status code returned, POST request can be redirected to an HTTP GET or POST.

As per HTTP 1.1 protocol reference, status codes 301 (Moved Permanently) and 302 (Found) allow the request method to be changed from POST to GET. The specification also defines the corresponding 307 (Temporary Redirect) and 308 (Permanent Redirect) status codes that don’t allow the request method to be changed from POST to GET.

Let’s look at the code for redirecting a post request to another post request:
