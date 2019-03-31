### HTTP协议及其应用

HTTP协议应该是互联网领域运用最广泛的一种协议，它的重要性无需多言，作为web开发人员，这是我们最应该掌握的技能，首先我们看看HTTP请求和响应长什么样子。

首先我们用Go首先一个web server

```
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("hello wolrd"))
	})
	log.Fatal(http.ListenAndServe(":19028", nil))
}

```



