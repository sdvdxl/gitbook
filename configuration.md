# Configuration

Configuration is a relative package `github.com/kataras/iris/config` 

>  No need to download it separately, it's downloaded automatically when you install Iris.

### Why?
I took this decision after a lot of thought and I ensure you that this is the best and easiest
architecture:

- change the configs without needing to re-write all of their fields.
	```go
	irisConfig := config.Iris { Profile: true, DisablePathCorrection: true }
	api := iris.New(irisConfig)
	```
- **easy to remember**: `iris` type takes `config.Iris`, sessions takes `config.Sessions`, `iris.Config().Render` is of type `config.Render`, `iris.Config().Render.Template` is the type `config.Template`, `Logger` takes `config.Logger` and so on...

- **easy to search & find out what features exists and what you can change**: just navigate to the config folder and open the type you want to learn about, for example `/iris.go` Iris' type configuration is in `/config/iris.go`

- **All structs that receive configurations are already set to sane defaults** , so for casual use you can ignore them, but if you ever need to check them, you can find their default configs by this pattern: for example `config.Template` has `config.DefaultTemplate()`, `config.Rest` has `config.DefaultRest()`, `config.Typescript()` has `config.DefaultTypescript()`. Note however that `config.Iris` uses `config.Default()`. Even the plugins have their own default configs, to make it easier for you. 

- Enables you to do this **without setting up a config yourself**: `iris.Config().Render.Template.Engine = config.PongoEngine` or `iris.Config().Render.Template.Pongo.Extensions = []string{".xhtml", ".html"}`.

- **(Advanced usage) merge configs**: 

```go
//...
import "github.com/kataras/iris/config"
//...
templateFromRoutine1 := config.DefaultTemplate()
//....
templateFromOthers := config.Template{ Directory: "views"}

templateConfig := templateFromRoutine1.MergeSingle(templateFromOthers)

iris.Config().Render.Template = templateConfig

```

Below you will find a list of the config structs.

## Search [All Configs](https://github.com/kataras/iris/tree/master/config)
```go

// Inside iris package
var (
	HTMLEngine  = config.HTMLEngine
	PongoEngine = config.PongoEngine
    MarkdownEngine = config.MarkdownEngine
	JadeEngine  = config.JadeEngine
	AmberEngine = config.AmberEngine

	DefaultEngine = config.DefaultEngine
	NoEngine      = config.NoEngine
	//

	NoLayout = config.NoLayout
)

type (
	// Iris configs for the station
	// All fields can be changed before server's listen except the PathCorrection field
	//
	// MaxRequestBodySize is the only options that can be changed after server listen -
	// using Config().MaxRequestBodySize = ...
	// Render's rest config can be changed after declaration but before server's listen -
	// using Config().Render.Rest...
	// Render's Template config can be changed after declaration but before server's listen -
	// using Config().Render.Template...
	// Sessions config can be changed after declaration but before server's listen -
	// using Config().Sessions...
	// and so on...
	Iris struct {
		// DisablePathCorrection corrects and redirects the requested path to the registed path
		// for example, if /home/ path is requested but no handler for this Route found,
		// then the Router checks if /home handler exists, if yes,
		// (permant)redirects the client to the correct path /home
		//
		// Default is false
		DisablePathCorrection bool

		// DisablePathEscape when is false then its escapes the path, the named parameters (if any).
		// Change to true it if you want something like this https://github.com/kataras/iris/issues/135 to work
		//
		// When do you need to Disable(true) it:
		// accepts parameters with slash '/'
		// Request: http://localhost:8080/details/Project%2FDelta
		// ctx.Param("project") returns the raw named parameter: Project%2FDelta
		// which you can escape it manually with net/url:
		// projectName, _ := url.QueryUnescape(c.Param("project").
		// Look here: https://github.com/kataras/iris/issues/135 for more
		//
		// Default is false
		DisablePathEscape bool

		// DisableBanner outputs the iris banner at startup
		//
		// Default is false
		DisableBanner bool

		// MaxRequestBodySize Maximum request body size.
		//
		// The server rejects requests with bodies exceeding this limit.
		//
		// By default request body size is -1, unlimited.
		MaxRequestBodySize int64

		// Profile set to true to enable web pprof (debug profiling)
		// Default is false, enabling makes available these 7 routes:
		// /debug/pprof/cmdline
		// /debug/pprof/profile
		// /debug/pprof/symbol
		// /debug/pprof/goroutine
		// /debug/pprof/heap
		// /debug/pprof/threadcreate
		// /debug/pprof/pprof/block
		Profile bool

		// ProfilePath change it if you want other url path than the default
		// Default is /debug/pprof , which means yourhost.com/debug/pprof
		ProfilePath string

		// Logger the configuration for the logger
		// Iris logs ONLY errors and the banner if enabled
		Logger Logger

		// Sessions the config for sessions
		// contains 3(three) properties
		// Provider: (look /sessions/providers)
		// Secret: cookie's name (string)
		// Life: cookie life (time.Duration)
		Sessions Sessions

		// Render contains the configs for template and rest configuration
		Render Render

		// Websocket contains the configs for Websocket's server integration
		Websocket Websocket

		// Mail contains the config for the mail sender service
		Mail Mail
	}

	// Render struct keeps organise all configuration about rendering, templates and rest currently.
	Render struct {
		// Template the configs for template
		Template Template
		// Rest configs for rendering.
		//
		// these options inside this config don't have any relation with the TemplateEngine
		// from github.com/kataras/iris/rest
		Rest Rest
	}
)

```

```go

type (
	// Rest is a struct for specifying configuration options for the rest.Render object.
	Rest struct {
		// Appends the given character set to the Content-Type header. Default is "UTF-8".
		Charset string
		// Gzip enable it if you want to render with gzip compression. Default is false
		Gzip bool
		// Outputs human readable JSON.
		IndentJSON bool
		// Outputs human readable XML. Default is false.
		IndentXML bool
		// Prefixes the JSON output with the given bytes. Default is false.
		PrefixJSON []byte
		// Prefixes the XML output with the given bytes.
		PrefixXML []byte
		// Unescape HTML characters "&<>" to their original values. Default is false.
		UnEscapeHTML bool
		// Streams JSON responses instead of marshalling prior to sending. Default is false.
		StreamingJSON bool
		// Disables automatic rendering of http.StatusInternalServerError when an error occurs. 
        // Default is false.
		DisableHTTPErrorRendering bool
        // MarkdownSanitize sanitizes the markdown. Default is false.
		MarkdownSanitize bool
	}

        EngineType int8

		Template struct {
		// contains common configs for both HTMLTemplate & Pongo
		Engine EngineType
		Gzip   bool
		IsDevelopment bool
		Directory     string
		Extensions    []string
		ContentType   string
		Charset       string
		Asset         func(name string) ([]byte, error)
		AssetNames    func() []string
		Layout        string

		HTMLTemplate HTMLTemplate // contains specific configs for  HTMLTemplate standard html/template
		Pongo        Pongo        // contains specific configs for pongo2
		// Markdown template engine it doesn't supports Layout & binding context
		Markdown Markdown // contains specific configs for markdown
		Jade     Jade     // contains specific configs for Jade
		Amber    Amber    // contains specific configs for Amber
	}

	HTMLTemplate struct {
		RequirePartials bool
		// Delims
		Left  string
		Right string
		// Funcs for HTMLTemplate html/template
		Funcs template.FuncMap
	}

	Pongo struct {
		// Filters for pongo2, map[name of the filter] the filter function . The filters are auto register
		Filters map[string]pongo2.FilterFunction
	}

	Markdown struct {
		Sanitize bool // if true then returns safe html, default is false
	}

    // Jade is using the same HTMLEngine's configs
	Jade HTMLTemplate

	Amber struct {
		// Funcs for the html/template result,
        // amber default funcs are not overrided so use it without worries
		Funcs template.FuncMap
	}
)
```

```go

var (
	universe time.Time // 0001-01-01 00:00:00 +0000 UTC
	// CookieExpireNever the default cookie's life for sessions, unlimited
	CookieExpireNever = universe
)

const (
	// DefaultCookieName the secret cookie's name for sessions
	DefaultCookieName        = "irissessionid"
	DefaultSessionGcDuration = time.Duration(2) * time.Hour
	// DefaultRedisNetwork the redis network option, "tcp"
	DefaultRedisNetwork = "tcp"
	// DefaultRedisAddr the redis address option, "127.0.0.1:6379"
	DefaultRedisAddr = "127.0.0.1:6379"
	// DefaultRedisIdleTimeout the redis idle timeout option, time.Duration(5) * time.Minute
	DefaultRedisIdleTimeout = time.Duration(5) * time.Minute
	// DefaultRedisMaxAgeSeconds the redis storage last parameter (SETEX), 31556926.0 (1 year)
	DefaultRedisMaxAgeSeconds = 31556926.0 //1 year

)

type (

	// Redis the redis configuration used inside sessions
	Redis struct {
		// Network "tcp"
		Network string
		// Addr "127.0.01:6379"
		Addr string
		// Password string .If no password then no 'AUTH'. Default ""
		Password string
		// If Database is empty "" then no 'SELECT'. Default ""
		Database string
		// MaxIdle 0 no limit
		MaxIdle int
		// MaxActive 0 no limit
		MaxActive int
		// IdleTimeout  time.Duration(5) * time.Minute
		IdleTimeout time.Duration
		// Prefix "myprefix-for-this-website". Default ""
		Prefix string
		// MaxAgeSeconds how much long the redis should keep the session in seconds. 
        // Default 31556926.0 (1 year)
		MaxAgeSeconds int
	}

	// Sessions the configuration for sessions
	// has 4 fields
	// first is the providerName (string) ["memory","redis"]
	// second is the cookieName, the session's name (string) ["mysessionsecretcookieid"]
	// third is the time which the client's cookie expires
	// forth is the gcDuration (time.Duration) 
    // when this time passes it removes the unused sessions from the memory until the user come back
	Sessions struct {
		// Provider string, usage iris.Config().Provider = "memory" or "redis".
        // If you wan to customize redis then import the package, and change it's config
		Provider string
		// Cookie string, the session's client cookie name, for example: "irissessionid"
		Cookie string
		//Expires the date which the cookie must expires. Default infinitive/unlimited life
		Expires time.Time
		//GcDuration every how much duration(GcDuration) 
        // the memory should be clear for unused cookies (GcDuration)
		//for example: time.Duration(2)*time.Hour. 
        // it will check every 2 hours if cookie hasn't be used for 2 hours,
		// deletes it from memory until the user comes back, 
        // then the session continue to work as it was
		//
		// Default 2 hours
		GcDuration time.Duration
	}
)
```

```go
type (
	// Logger contains the full configuration options fields for the Logger
	Logger struct {
		// Out the (file) writer which the messages/logs will printed to
		// Default is os.Stdout
		Out *os.File
		// Prefix the prefix for each message
		// Default is ""
		Prefix string
		// Disabled default is false
		Disabled bool

		// foreground colors single SGR Code

		// ColorFgDefault the foreground color for the normal message bodies
		ColorFgDefault int
		// ColorFgInfo the foreground  color for info messages
		ColorFgInfo int
		// ColorFgSuccess the foreground color for success messages
		ColorFgSuccess int
		// ColorFgWarning the foreground color for warning messages
		ColorFgWarning int
		// ColorFgDanger the foreground color for error messages
		ColorFgDanger int
		// OtherFgColor the foreground color for the rest of the message types
		ColorFgOther int

		// background colors single SGR Code

		// ColorBgDefault the background color for the normal messages
		ColorBgDefault int
		// ColorBgInfo the background  color for info messages
		ColorBgInfo int
		// ColorBgSuccess the background color for success messages
		ColorBgSuccess int
		// ColorBgWarning the background color for warning messages
		ColorBgWarning int
		// ColorBgDanger the background color for error messages
		ColorBgDanger int
		// OtherFgColor the background color for the rest of the message types
		ColorBgOther int

		// banners are the force printed/written messages, doesn't care about Disabled field
		
        // ColorFgBanner the foreground color for the banner
		ColorFgBanner int
	}
)
```

```go
type (
	// Tsconfig the struct for tsconfig.json
	Tsconfig struct {
		CompilerOptions CompilerOptions `json:"compilerOptions"`
		Exclude         []string        `json:"exclude"`
	}

	// CompilerOptions contains all the compiler options used by the tsc (typescript compiler)
	CompilerOptions struct {
		Declaration                      bool   `json:"declaration"`
		Module                           string `json:"module"`
		Target                           string `json:"target"`
		Watch                            bool   `json:"watch"`
		Charset                          string `json:"charset"`
		Diagnostics                      bool   `json:"diagnostics"`
		EmitBOM                          bool   `json:"emitBOM"`
		EmitDecoratorMetadata            bool   `json:"emitDecoratorMetadata"`
		ExperimentalDecorators           bool   `json:"experimentalDecorators"`
		InlineSourceMap                  bool   `json:"inlineSourceMap"`
		InlineSources                    bool   `json:"inlineSources"`
		IsolatedModules                  bool   `json:"isolatedModules"`
		Jsx                              string `json:"jsx"`
		ReactNamespace                   string `json:"reactNamespace"`
		ListFiles                        bool   `json:"listFiles"`
		Locale                           string `json:"locale"`
		MapRoot                          string `json:"mapRoot"`
		ModuleResolution                 string `json:"moduleResolution"`
		NewLine                          string `json:"newLine"`
		NoEmit                           bool   `json:"noEmit"`
		NoEmitOnError                    bool   `json:"noEmitOnError"`
		NoEmitHelpers                    bool   `json:"noEmitHelpers"`
		NoImplicitAny                    bool   `json:"noImplicitAny"`
		NoLib                            bool   `json:"noLib"`
		NoResolve                        bool   `json:"noResolve"`
		SkipDefaultLibCheck              bool   `json:"skipDefaultLibCheck"`
		OutDir                           string `json:"outDir"`
		OutFile                          string `json:"outFile"`
		PreserveConstEnums               bool   `json:"preserveConstEnums"`
		Pretty                           bool   `json:"pretty"`
		RemoveComments                   bool   `json:"removeComments"`
		RootDir                          string `json:"rootDir"`
		SourceMap                        bool   `json:"sourceMap"`
		SourceRoot                       string `json:"sourceRoot"`
		StripInternal                    bool   `json:"stripInternal"`
		SuppressExcessPropertyErrors     bool   `json:"suppressExcessPropertyErrors"`
		SuppressImplicitAnyIndexErrors   bool   `json:"suppressImplicitAnyIndexErrors"`
		AllowUnusedLabels                bool   `json:"allowUnusedLabels"`
		NoImplicitReturns                bool   `json:"noImplicitReturns"`
		NoFallthroughCasesInSwitch       bool   `json:"noFallthroughCasesInSwitch"`
		AllowUnreachableCode             bool   `json:"allowUnreachableCode"`
		ForceConsistentCasingInFileNames bool   `json:"forceConsistentCasingInFileNames"`
		AllowSyntheticDefaultImports     bool   `json:"allowSyntheticDefaultImports"`
		AllowJs                          bool   `json:"allowJs"`
		NoImplicitUseStrict              bool   `json:"noImplicitUseStrict"`
	}

	Typescript struct {
		Bin      string
		Dir      string
		Ignore   string
		Tsconfig Tsconfig
		Editor   Editor
	}
)

```

```go

var (
	// DefaultUsername used for default (basic auth)
    // username in IrisControl's & Editor's default configuration
	DefaultUsername = "iris"
	// DefaultPassword used for default (basic auth) 
    // password in IrisControl's & Editor's default configuration
	DefaultPassword = "admin!123"
)

// IrisControl the options which iris control needs
// contains the port (int) and authenticated users with their passwords (map[string]string)
type IrisControl struct {
	// Port the port
	Port int
	// Users the authenticated users, [username]password
	Users map[string]string
}
```


```go
type Editor struct {
	// Host if empty used the iris server's host
	Host string
	// Port if 0 4444
	Port int
	// WorkingDir if empty "./"
	WorkingDir string
	// Useranme if empty iris
	Username string
	// Password if empty admin!123
	Password string
}
```

```go
type Websocket struct {
	// WriteTimeout time allowed to write a message to the connection.
	// Default value is 10 * time.Second
	WriteTimeout time.Duration
	// PongTimeout allowed to read the next pong message from the connection
	// Default value is 60 * time.Second
	PongTimeout time.Duration
	// PingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
	// Default value is (PongTimeout * 9) / 10
	PingPeriod time.Duration
	// MaxMessageSize max message size allowed from connection
	// Default value is 1024
	MaxMessageSize int
	// Endpoint is the path which the websocket server will listen for clients/connections
	// Default value is empty string, if you don't set it the Websocket server is disabled.
	Endpoint string
    // Headers  the response headers before upgrader
	// Default is empty
	Headers map[string]string
}

```


```go
// Mail keeps the configs for mail sender service
type Mail struct {
	// Host is the server mail host, IP or address
	Host string
	// Port is the listening port
	Port int
	// Username is the auth username@domain.com for the sender
	Username string
	// Password is the auth password for the sender
	Password string
    // FromAlias is the from part, if empty this is the first part before @ from the Username field
	FromAlias string
    // UseCommand enable it if you want to send e-mail with the mail command  instead of smtp
	//
	// Host,Port & Password will be ignored
	// ONLY FOR UNIX
	UseCommand bool
}
```
