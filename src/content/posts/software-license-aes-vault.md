---
title: "Time-Limited Licenses"
description: "meta description"
date: 2024-07-27T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["software license", "os"]
authors: ["Balaji Balasundaram"]
tags: ["software", "license", "aes", "vault"]
draft: false
---

#### 1. Set Up HashiCorp Vault
First, ensure that HashiCorp Vault is installed and running. You can follow the official documentation to set it up.

#### 2. Initialize and Unseal Vault
Initialize and unseal Vault if it is not already done. You can do this using the Vault CLI:

```sh
vault operator init
vault operator unseal
vault login <token>
```

#### 3. Store Encryption Key in Vault
Store the encryption key in Vault:

```sh
vault kv put secret/encryption_key value="a very very very very secret key"
```
#### 4. Go Code to Integrate with Vault
Use the Vault API in your Go code to fetch the encryption key and use it for encryption and decryption.

Install Vault API Client
First, install the Vault API client for Go:

```sh
go get github.com/hashicorp/vault/api
```
Go Code Example
Here’s a complete example that integrates with Vault to fetch the encryption key, generate secure license keys, encrypt them using AES-256, store them in a SQLite database, and validate the encrypted license data:

```go
package main

import (
	"context"
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"database/sql"
	"encoding/base64"
	"encoding/hex"
	"fmt"
	"io"
	"log"
	"time"

	"github.com/hashicorp/vault/api"
	_ "github.com/mattn/go-sqlite3"
)

var encryptionKey []byte

func init() {
	var err error
	encryptionKey, err = getEncryptionKeyFromVault()
	if err != nil {
		log.Fatal("Error fetching encryption key from Vault:", err)
	}
}

func getEncryptionKeyFromVault() ([]byte, error) {
	config := api.DefaultConfig()
	client, err := api.NewClient(config)
	if err != nil {
		return nil, err
	}

	client.SetToken("your-vault-token") // Replace with your Vault token

	secret, err := client.KVv2("secret").Get(context.Background(), "encryption_key")
	if err != nil {
		return nil, err
	}

	key, ok := secret.Data["value"].(string)
	if !ok {
		return nil, fmt.Errorf("encryption key not found in Vault")
	}

	if len(key) != 32 {
		return nil, fmt.Errorf("encryption key must be 32 bytes long")
	}

	return []byte(key), nil
}

func generateSecureLicenseKey() (string, error) {
	bytes := make([]byte, 32) // 256 bits
	_, err := rand.Read(bytes)
	if err != nil {
		return "", err
	}
	return base64.URLEncoding.EncodeToString(bytes), nil
}

func encrypt(data string) (string, error) {
	block, err := aes.NewCipher(encryptionKey)
	if err != nil {
		return "", err
	}

	ciphertext := make([]byte, aes.BlockSize+len(data))
	iv := ciphertext[:aes.BlockSize]
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return "", err
	}

	stream := cipher.NewCFBEncrypter(block, iv)
	stream.XORKeyStream(ciphertext[aes.BlockSize:], []byte(data))

	return hex.EncodeToString(ciphertext), nil
}

func decrypt(encrypted string) (string, error) {
	ciphertext, _ := hex.DecodeString(encrypted)
	block, err := aes.NewCipher(encryptionKey)
	if err != nil {
		return "", err
	}

	iv := ciphertext[:aes.BlockSize]
	ciphertext = ciphertext[aes.BlockSize:]

	stream := cipher.NewCFBDecrypter(block, iv)
	stream.XORKeyStream(ciphertext, ciphertext)

	return string(ciphertext), nil
}

func setupDatabase() *sql.DB {
	db, err := sql.Open("sqlite3", "./licenses.db")
	if err != nil {
		log.Fatal(err)
	}

	// Create licenses table if it doesn't exist
	sqlStmt := `
	CREATE TABLE IF NOT EXISTS licenses (
		id INTEGER PRIMARY KEY AUTOINCREMENT,
		license_key TEXT NOT NULL,
		issue_date DATETIME NOT NULL,
		duration_days INTEGER NOT NULL
	);
	`
	_, err = db.Exec(sqlStmt)
	if err != nil {
		log.Fatalf("%q: %s\n", err, sqlStmt)
	}

	return db
}

func storeLicense(db *sql.DB, licenseKey, issueDate string, durationDays int) error {
	encryptedKey, err := encrypt(licenseKey)
	if err != nil {
		return err
	}

	_, err = db.Exec("INSERT INTO licenses (license_key, issue_date, duration_days) VALUES (?, ?, ?)",
		encryptedKey, issueDate, durationDays)
	return err
}

func validateEncryptedLicense(db *sql.DB, licenseKey string) bool {
	var encryptedKey string
	var issueDate string
	var durationDays int

	err := db.QueryRow("SELECT license_key, issue_date, duration_days FROM licenses WHERE license_key = ?", licenseKey).Scan(&encryptedKey, &issueDate, &durationDays)
	if err != nil {
		if err == sql.ErrNoRows {
			fmt.Println("License key not found")
			return false
		}
		log.Fatal(err)
	}

	decryptedKey, err := decrypt(encryptedKey)
	if err != nil {
		fmt.Println("Error decrypting license key:", err)
		return false
	}

	if decryptedKey != licenseKey {
		fmt.Println("License key mismatch")
		return false
	}

	// Validate license expiration logic here...
	issuedDate, err := time.Parse("2006-01-02", issueDate)
	if err != nil {
		fmt.Println("Error parsing issue date:", err)
		return false
	}

	expiryDate := issuedDate.AddDate(0, 0, durationDays)
	if time.Now().After(expiryDate) {
		fmt.Println("License has expired")
		return false
	}

	fmt.Println("License is valid")
	return true
}

func main() {
	db := setupDatabase()
	defer db.Close()

	// Example of generating and storing a new license
	licenseKey, err := generateSecureLicenseKey()
	if err != nil {
		fmt.Println("Error generating license key:", err)
		return
	}

	issueDate := time.Now().Format("2006-01-02")
	durationDays := 30

	err = storeLicense(db, licenseKey, issueDate, durationDays)
	if err != nil {
		fmt.Println("Error storing license:", err)
		return
	}

	fmt.Println("License stored successfully")

	// Example of validating an existing license
	if validateEncryptedLicense(db, licenseKey) {
		fmt.Println("Access granted")
	} else {
		fmt.Println("Access denied")
	}
}
```
#### Summary
- Set Up Vault: Initialize and unseal Vault, then store the encryption key securely.
- Fetch Encryption Key from Vault: Use the Vault API client in Go to fetch the encryption key securely.
- Secure Encryption and Decryption: Use AES-256 for encrypting and decrypting license data.
- Store and Validate Licenses: Encrypt license keys before storing them in the database and decrypt them during validation.

By integrating Vault for key management, you enhance the security of your software licensing system and ensure that encryption keys are managed securely.


