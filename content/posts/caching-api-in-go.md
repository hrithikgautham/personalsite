---
title: "Caching API response in Go"
date: 2021-05-21T22:02:14+05:30
draft: false
tags: [Golang, tech, caching, API]
---

Github link - https://github.com/bsidio/traderbase_go

TL;DR -  Reduced Response time from seconds to milliseconds using In memory Cache 

Disclaimer - This ain't a deep technical post. I am in the process of learning Go. My method to learn anything is learn while building. So there might be more optimal solutions out there. But this one fits my use case.

When building any applications, I always build from the end user point of view. What if I am the user of my website and What are my expectations when using the site. The performance and UI are the two important factors for me. No matter how performative the backend is if UI is clunky I will not use it. One may have a great UI but if it takes forever to load can affect user retentions. While building any product we have to make sure both frontend and backend are as optimal as possible. 

When building tr.bsid.io (A Financial analysis platform). We do complex queries to retrieve data from multiple tables sometimes with join queries and fetch hell lot of data. I am in the process of rewriting the backend in Go which is currently in NodeJs. With the new code I had the opportunity to implement caching of API Respone.

Ok enough talk, Show me the code!


        package http

        import (
            "fmt"
            "net/http"
            "net/http/httptest"
            "time"

            "github.com/bsidio/traderbase_go/internal/transport/cache/memory"
        )

        type Storage interface {
            Get(key string) []byte
            Set(key string, content []byte, duration time.Duration)
        }

        var storage Storage

        func init() {

            storage = memory.NewStorage()

        }

      

        func (h *Handler) cached(handler http.HandlerFunc) http.HandlerFunc {
            return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
                duration := "24h"

                content := storage.Get(r.RequestURI)
                if content != nil {
                    w.Header().Add("Content-Type", "application/json")

                    w.Write(content)
                } else {
                    c := httptest.NewRecorder()
                    handler(c, r)

                    for k, v := range c.HeaderMap {
                        w.Header()[k] = v
                    }

                    w.WriteHeader(c.Code)
                    content := c.Body.Bytes()

                    if d, err := time.ParseDuration(duration); err == nil {
                        storage.Set(r.RequestURI, content, d)
                    } else {
                        fmt.Printf("Page not cached. err: %s\n", err)
                    }

                    w.Write(content)
                }

            })
        }


cache.go ->

        package memory

        import (
            "sync"
            "time"
        )

        // Item is a cached reference
        type Item struct {
            Content    []byte
            Expiration int64
        }

        // Expired returns true if the item has expired.
        func (item Item) Expired() bool {
            if item.Expiration == 0 {
                return false
            }
            return time.Now().UnixNano() > item.Expiration
        }

        //Storage mecanism for caching strings in memory
        type Storage struct {
            items map[string]Item
            mu    *sync.RWMutex
        }

        //NewStorage creates a new in memory storage
        func NewStorage() *Storage {
            return &Storage{
                items: make(map[string]Item),
                mu:    &sync.RWMutex{},
            }
        }

        //Get a cached content by key
        func (s Storage) Get(key string) []byte {
            s.mu.RLock()
            defer s.mu.RUnlock()
            item := s.items[key]
            if item.Expired() {
                delete(s.items, key)
                return nil
            }
            return item.Content
        }

        //Set a cached content by key
        func (s Storage) Set(key string, content []byte, duration time.Duration) {
            //fmt.Println(content)

            s.mu.Lock()
            defer s.mu.Unlock()

            s.items[key] = Item{
                Content:    content,
                Expiration: time.Now().Add(duration).UnixNano(),
            }
            //	fmt.Println(s.items[key])
        }

handler.go ->
Cached is an http middleware that runs before the http handler and returns the content straight away if the page is already cached. If it’s not, the handler is executed and its body is cached for a given period of time. Because it’s a middleware, it’s really easy to enable and disable it for certain routes. Keep reading for a concrete example.

        package http

        import (
            "encoding/json"
            "fmt"
            "net/http"

            "github.com/bsidio/traderbase_go/internal/data"

            "github.com/gorilla/mux"
        )

        //Handler - stores pointer to service
        type Handler struct {
            Router  *mux.Router
            Service *data.Service
        }

        //ad object to store response
        type Response struct {
            Message string
            Error   string
        }

        //Returns pointer to  Handler Struct
        func NewHandler(service *data.Service) *Handler {
            return &Handler{
                Service: service,
            }
        }

        //setup all routers to our application
        func (h *Handler) SetupRoutes() {
            fmt.Println("Setting up routes")
            h.Router = mux.NewRouter()
            h.Router.HandleFunc("/api/health", func(w http.ResponseWriter, r *http.Request) {
                //w.Header().Set("Contect-Type", "application/json; charset=UTF-8")
                w.Header().Add("Content-Type", "application/json")
                w.WriteHeader(http.StatusOK)
                if err := json.NewEncoder(w).Encode(Response{Message: "I am Alive"}); err != nil {
                    panic(err)
                }
            })
            h.Router.HandleFunc("/api/earnings/amc/{ticker}", h.cached(h.GetAmcEarningsData)).Methods("GET")
            h.Router.HandleFunc("/api/earnings/bmo/{ticker}", h.cached(h.GetBmoEarningsData)).Methods("GET")
            h.Router.HandleFunc("/api/uploaded/{ticker}", h.cached(h.GetUploadedData)).Methods("GET")
            h.Router.HandleFunc("/api/uploaded/options/{ticker}", h.cached(h.GetUploadedOptionsData)).Methods("GET")
            h.Router.HandleFunc("/api/uploaded/sector/{ticker}", h.cached(h.GetUploadedSectorData)).Methods("GET")
            h.Router.HandleFunc("/api/uploaded/industry/{ticker}", h.cached(h.GetUploadedIndData)).Methods("GET")
            h.Router.HandleFunc("/api/uploaded/subindustry/{ticker}", h.cached(h.GetUploadedSubIndData)).Methods("GET")
            h.Router.HandleFunc("/api/insider/{ticker}", h.cached(h.GetInsiderData)).Methods("GET")
            h.Router.HandleFunc("/api/iex/stats/{ticker}", h.cached(h.GetIexStatsData)).Methods("GET")
            h.Router.HandleFunc("/api/iex/previous/{ticker}", h.cached(h.GetIexPrevData)).Methods("GET")
            h.Router.HandleFunc("/api/iex/history/{time}/{ticker}", h.cached(h.GetIexchartData)).Methods("GET")
            h.Router.HandleFunc("/api/iex/history/chart/{time}/{ticker}", h.cached(h.GetIexchartDataOnly)).Methods("GET")
            h.Router.HandleFunc("/api/coverage/{ticker}", h.cached(h.GetCoverageData)).Methods("GET")
            h.Router.HandleFunc("/api/stocks/list/{type}/{week}", h.cached(h.GetStockListData)).Methods("GET")

        }

        func (h *Handler) sendErrorResponse(w http.ResponseWriter, message string, err error) {
            w.WriteHeader(http.StatusInternalServerError)
            if err := json.NewEncoder(w).Encode(Response{Message: message, Error: err.Error()}); err != nil {
                panic(err)
            }
        }

Results -

Before ->
{{< figure src="/static/images/cachebefore.png"  >}}
After ->
{{< figure src="/static/images/cacheafter.png"  >}}
