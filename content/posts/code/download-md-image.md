---
title: "golang程序下载图片到本地"
date: 2021-02-16T22:13:45+08:00
tags: ["代码片段"]
categories: ["IT"]
---

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"regexp"
	"strings"

	//"io"
	"os"
	//"sync"
)

var downloadDir = "./upload/iloli"

func checkDownloadDir() {
	_, err := os.Stat(downloadDir)

	if os.IsNotExist(err) {
		os.MkdirAll(downloadDir, 0777)
		os.Chmod(downloadDir, 0777)
	}
}

func main() {
	checkDownloadDir()
	l := getMDList(`./content`)
	allImages := map[string]bool{}
	for _, f := range l {
		images := parseImage(f)
		for _, i := range images {
			allImages[i] = true
		}
	}
	/*
	   wg := &sync.WaitGroup{}
	   wg.Add(len(allImages))
	   wg.Wait()
	*/
	for k, _ := range allImages {
		downloadImage(k)
	}

}

func downloadImage(imgURL string) {

	//defer wg.Done()
	imgPath := strings.Replace(imgURL, "https://i.loli.net/", "", -1)

	ss := strings.Split(imgPath, "/")
	imageDir := fmt.Sprintf("%s/%s", downloadDir, strings.Join(ss[0:(len(ss)-1)], "/"))
	imageFullPath := fmt.Sprintf("%s/%s", downloadDir, imgPath)

	if _, err := os.Stat(imageFullPath); os.IsNotExist(err) {
		fmt.Println("download start：", imgURL)
		res, errX := http.Get(imgURL)
		if errX != nil {
			fmt.Println("downloadImage error:", imgURL, err)
			return
		}

		if _, err0 := os.Stat(imageDir); os.IsNotExist(err0) {
			os.MkdirAll(imageDir, 0777)
			os.Chmod(imageDir, 0777)
		}

		if body, err1 := ioutil.ReadAll(res.Body); err1 == nil {
			if err2 := ioutil.WriteFile(imageFullPath, body, os.ModePerm); err2 == nil {
				fmt.Println("finish:", imageFullPath)
			} else {
				fmt.Println(err2)
			}

		}
	} else {
		fmt.Println("已存在，跳过...")
	}

}

func parseImage(f string) []string {
	content, _ := ioutil.ReadFile(f)
	eg, err := regexp.Compile(`https:\/\/i.loli.net\/\d{4}\/\d{2}/\d{2}\/\S+.png`)
	if err != nil {
		panic(err)
	}

	images := eg.FindAllString(string(content), -1)
	return images
}

func getMDList(folder string) []string {

	files, _ := ioutil.ReadDir(folder)
	retData := []string{}
	for _, file := range files {
		if file.IsDir() {
			tmp := getMDList(fmt.Sprintf("%s/%s", folder, file.Name()))
			retData = append(retData, tmp...)
		} else {
			if strings.HasSuffix(file.Name(), ".md") {
				retData = append(retData, fmt.Sprintf("%s/%s", folder, file.Name()))
			}

		}
	}
	return retData
}

```

