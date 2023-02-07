# Go Soap [![Build Status](https://travis-ci.org/tiaguinho/gosoap.svg?branch=master)](https://travis-ci.org/tiaguinho/gosoap) [![GoDoc](https://godoc.org/github.com/tiaguinho/gosoap?status.png)](https://godoc.org/github.com/tiaguinho/gosoap) [![Go Report Card](https://goreportcard.com/badge/github.com/tiaguinho/gosoap)](https://goreportcard.com/report/github.com/tiaguinho/gosoap) [![Coverage Status](https://coveralls.io/repos/github/tiaguinho/gosoap/badge.svg?branch=master)](https://coveralls.io/github/tiaguinho/gosoap?branch=master) [![patreon](https://img.shields.io/badge/patreon-donate-yellow.svg)](https://www.patreon.com/temporin)

package to help with SOAP integrations (client)

### Install

```bash
go get github.com/tinybear1976/gosoap
```

### Examples

#### 用例

在前作基础上简单修改增加了 User-Agent 的修改，以及 soap1.1 ，1.2 版本的切换

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/tinybear1976/gosoap"
)

func main() {
	getIp2()
}

func getIp2() {
	wsdl_uri := "http://wsgeoip.lavasoft.com/ipservice.asmx?WSDL"
	userAgent := "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
	httpClient := &http.Client{
		Timeout: 1000 * 1000 * time.Millisecond,
	}
	soap, err := gosoap.SoapClient(wsdl_uri, httpClient)
	soap.SetSoapVersion(2)
	soap.SetUserAgent(userAgent)
	if err != nil {
		log.Fatalf("SoapClient error: %s", err)
	}

	action := "GetIpLocation"
	params := gosoap.Params{
		"sIp": "4.4.4.4",
	}
	res, err := soap.Call(action, params)
	if err != nil {
		log.Fatalf("Call error: %s", err)
	}

	log.Println(string(res.Body))
	var r GetIpLocationResult
	res.Unmarshal(&r)
	log.Println(r)
}

type GetIpLocationResult struct {
	Data string `xml:"GetIpLocationResult"`
}

func getIp() {
	wsdl_uri := "http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl"
	userAgent := "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
	httpClient := &http.Client{
		Timeout: 1500 * time.Millisecond,
	}
	soap, err := gosoap.SoapClient(wsdl_uri, httpClient)
	soap.SetSoapVersion(2)
	soap.SetUserAgent(userAgent)
	if err != nil {
		log.Fatalf("SoapClient error: %s", err)
	}

	action := "getCountryCityByIp"
	params := gosoap.Params{
		"theIpAddress": "4.4.4.4",
	}
	res, err := soap.Call(action, params)
	if err != nil {
		log.Fatalf("Call error: %s", err)
	}

	log.Println(string(res.Body))
	var r GetCountryCityByIpResult
	res.Unmarshal(&r)
	log.Println(r)
}

type GetCountryCityByIpResult struct {
	ArrayString []string `xml:"getCountryCityByIpResult>string"`
}

func getStationNames() {
	wsdl_uri := "http://www.webxml.com.cn/webservices/TrainTimeWebService.asmx?wsdl"
	httpClient := &http.Client{
		Timeout: 1500 * time.Millisecond,
	}
	soap, err := gosoap.SoapClient(wsdl_uri, httpClient)
	if err != nil {
		log.Fatalf("SoapClient error: %s", err)
	}

	// Use gosoap.ArrayParams to support fixed position params
	params := gosoap.Params{}
	// action := "getVersionTime"
	action := "getStationName"
	res, err := soap.Call(action, params)
	if err != nil {
		log.Fatalf("Call error: %s", err)
	}

	log.Println(string(res.Body))
	var r GetStationNameResultResponse
	res.Unmarshal(&r)
	log.Println(r)
}

// hold the Soap response
type GetStationNameResultResponse struct {
	// StationNames []StationName `xml:"getStationNameResult>string"`
	StationNames []string `xml:"getStationNameResult>string"`
}

// type StationName struct {
// 	Name string `xml:string`
// }
```

#### Basic use

```go
package main

import (
	"encoding/xml"
	"log"
	"net/http"
	"time"

	"github.com/tiaguinho/gosoap"
)

// GetIPLocationResponse will hold the Soap response
type GetIPLocationResponse struct {
	GetIPLocationResult string `xml:"GetIpLocationResult"`
}

// GetIPLocationResult will
type GetIPLocationResult struct {
	XMLName xml.Name `xml:"GeoIP"`
	Country string   `xml:"Country"`
	State   string   `xml:"State"`
}

var (
	r GetIPLocationResponse
)

func main() {
	httpClient := &http.Client{
		Timeout: 1500 * time.Millisecond,
	}
	soap, err := gosoap.SoapClient("http://wsgeoip.lavasoft.com/ipservice.asmx?WSDL", httpClient)
	if err != nil {
		log.Fatalf("SoapClient error: %s", err)
	}

	// Use gosoap.ArrayParams to support fixed position params
	params := gosoap.Params{
		"sIp": "8.8.8.8",
	}

	res, err := soap.Call("GetIpLocation", params)
	if err != nil {
		log.Fatalf("Call error: %s", err)
	}

	res.Unmarshal(&r)

	// GetIpLocationResult will be a string. We need to parse it to XML
	result := GetIPLocationResult{}
	err = xml.Unmarshal([]byte(r.GetIPLocationResult), &result)
	if err != nil {
		log.Fatalf("xml.Unmarshal error: %s", err)
	}

	if result.Country != "US" {
		log.Fatalf("error: %+v", r)
	}

	log.Println("Country: ", result.Country)
	log.Println("State: ", result.State)
}
```

#### Set Custom Envelope Attributes

```go
package main

import (
	"encoding/xml"
	"log"
	"net/http"
	"time"

	"github.com/tiaguinho/gosoap"
)

// GetIPLocationResponse will hold the Soap response
type GetIPLocationResponse struct {
	GetIPLocationResult string `xml:"GetIpLocationResult"`
}

// GetIPLocationResult will
type GetIPLocationResult struct {
	XMLName xml.Name `xml:"GeoIP"`
	Country string   `xml:"Country"`
	State   string   `xml:"State"`
}

var (
	r GetIPLocationResponse
)

func main() {
	httpClient := &http.Client{
		Timeout: 1500 * time.Millisecond,
	}
	// set custom envelope
    gosoap.SetCustomEnvelope("soapenv", map[string]string{
		"xmlns:soapenv": "http://schemas.xmlsoap.org/soap/envelope/",
		"xmlns:tem": "http://tempuri.org/",
    })

	soap, err := gosoap.SoapClient("http://wsgeoip.lavasoft.com/ipservice.asmx?WSDL", httpClient)
	if err != nil {
		log.Fatalf("SoapClient error: %s", err)
	}

	// Use gosoap.ArrayParams to support fixed position params
	params := gosoap.Params{
		"sIp": "8.8.8.8",
	}

	res, err := soap.Call("GetIpLocation", params)
	if err != nil {
		log.Fatalf("Call error: %s", err)
	}

	res.Unmarshal(&r)

	// GetIpLocationResult will be a string. We need to parse it to XML
	result := GetIPLocationResult{}
	err = xml.Unmarshal([]byte(r.GetIPLocationResult), &result)
	if err != nil {
		log.Fatalf("xml.Unmarshal error: %s", err)
	}

	if result.Country != "US" {
		log.Fatalf("error: %+v", r)
	}

	log.Println("Country: ", result.Country)
	log.Println("State: ", result.State)
}
```
