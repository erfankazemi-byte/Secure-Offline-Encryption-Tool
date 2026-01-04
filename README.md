package main

import (
	"crypto/rand"
	"encoding/base64"
	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/container"
	"fyne.io/fyne/v2/dialog"
	"fyne.io/fyne/v2/theme"
	"fyne.io/fyne/v2/widget"
	"golang.org/x/crypto/argon2"
	"crypto/aes"
	"crypto/cipher"
)

type Config struct {
	Time    uint32
	Memory  uint32
	Threads uint8
}

var presets = map[string]Config{
	"High (Max Security)": {Time: 3, Memory: 1024 * 1024, Threads: 4}, // 1 GiB
	"Medium (Balanced)":   {Time: 3, Memory: 64 * 1024, Threads: 4},   // 64 MiB
	"Low (Fast)":          {Time: 2, Memory: 19 * 1024, Threads: 1},   // 19 MiB
}

func deriveKey(password string, salt []byte, config Config) []byte {
	return argon2.IDKey([]byte(password), salt, config.Time, config.Memory, config.Threads, 32)
}

func encrypt(plaintext, password, preset string) (string, error) {
	config := presets[preset]
	salt := make([]byte, 16)
	if _, err := rand.Read(salt); err != nil {
		return "", err
	}

	key := deriveKey(password, salt, config)
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	gcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	nonce := make([]byte, gcm.NonceSize())
	if _, err = rand.Read(nonce); err != nil {
		return "", err
	}

	ciphertext := gcm.Seal(nonce, nonce, []byte(plaintext), nil)
	blob := append(salt, ciphertext...)
	return base64.URLEncoding.EncodeToString(blob), nil
}

func decrypt(encrypted, password, preset string) (string, error) {
	config := presets[preset]
	data, err := base64.URLEncoding.DecodeString(encrypted)
	if err != nil {
		return "", err
	}

	if len(data) < 16+12 {
		return "", aes.KeySizeError(32)
	}

	salt := data[:16]
	ciphertext := data[16:]

	key := deriveKey(password, salt, config)
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	gcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	nonceSize := gcm.NonceSize()
	if len(ciphertext) < nonceSize {
		return "", err
	}

	nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
	plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
	if err != nil {
		return "", err
	}

	return string(plaintext), nil
}

func main() {
	a := app.NewWithID("org.example.secureencrypt")
	a.Settings().SetTheme(theme.DarkTheme())

	w := a.NewWindow("Secure Encrypt - Offline SMS Tool")
	w.Resize(fyne.NewSize(700, 800))
	w.CenterOnScreen()

	tabs := container.NewAppTabs()

	// Encrypt Tab
	encryptContent := container.NewVBox()

	msgEntry := widget.NewMultiLineEntry()
	msgEntry.SetPlaceHolder("Enter message (keep short for SMS)")

	passEntry := widget.NewPasswordEntry()
	passEntry.SetPlaceHolder("Strong password")

	presetSelect := widget.NewSelect([]string{"High (Max Security)", "Medium (Balanced)", "Low (Fast)"}, nil)
	presetSelect.SetSelected("High (Max Security)")

	resultEntry := widget.NewMultiLineEntry()
	resultEntry.Disable()

	encryptBtn := widget.NewButton("🔒 Encrypt", func() {
		text := msgEntry.Text
		pwd := passEntry.Text
		preset := presetSelect.Selected

		if text == "" || pwd == "" {
			dialog.ShowInformation("Error", "Message and password required!", w)
			return
		}

		encrypted, err := encrypt(text, pwd, preset)
		if err != nil {
			dialog.ShowError(err, w)
			return
		}

		resultEntry.SetText(encrypted)
		dialog.ShowInformation("Success", "Encrypted! Copy and send via SMS.", w)
	})

	copyBtn := widget.NewButton("📋 Copy to Clipboard", func() {
		if resultEntry.Text != "" {
			w.Clipboard().SetContent(resultEntry.Text)
			dialog.ShowInformation("Copied", "Encrypted text copied to clipboard!", w)
		}
	})

	encryptContent.Add(widget.NewLabel("Message:"))
	encryptContent.Add(msgEntry)
	encryptContent.Add(widget.NewLabel("Password:"))
	encryptContent.Add(passEntry)
	encryptContent.Add(widget.NewLabel("Security Level:"))
	encryptContent.Add(presetSelect)
	encryptContent.Add(encryptBtn)
	encryptContent.Add(widget.NewLabel("Encrypted Output:"))
	encryptContent.Add(resultEntry)
	encryptContent.Add(copyBtn)

	tabs.Append(container.NewTabItem("🔒 Encrypt", container.NewScroll(encryptContent)))

	// Decrypt Tab
	decryptContent := container.NewVBox()

	encEntry := widget.NewMultiLineEntry()
	encEntry.SetPlaceHolder("Paste encrypted Base64 text")

	decPassEntry := widget.NewPasswordEntry()

	decPresetSelect := widget.NewSelect([]string{"High (Max Security)", "Medium (Balanced)", "Low (Fast)"}, nil)
	decPresetSelect.SetSelected("High (Max Security)")

	decResultEntry := widget.NewMultiLineEntry()
	decResultEntry.Disable()

	decryptBtn := widget.NewButton("🔓 Decrypt", func() {
		enc := encEntry.Text
		pwd := decPassEntry.Text
		preset := decPresetSelect.Selected

		if enc == "" || pwd == "" {
			dialog.ShowInformation("Error", "Encrypted text and password required!", w)
			return
		}

		plaintext, err := decrypt(enc, pwd, preset)
		if err != nil {
			dialog.ShowError(err, w)
			return
		}

		decResultEntry.SetText(plaintext)
		dialog.ShowInformation("Success", "Message decrypted!", w)
	})

	decryptContent.Add(widget.NewLabel("Encrypted Text:"))
	decryptContent.Add(encEntry)
	decryptContent.Add(widget.NewLabel("Password:"))
	decryptContent.Add(decPassEntry)
	decryptContent.Add(widget.NewLabel("Security Level:"))
	decryptContent.Add(decPresetSelect)
	decryptContent.Add(decryptBtn)
	decryptContent.Add(widget.NewLabel("Decrypted Message:"))
	decryptContent.Add(decResultEntry)

	tabs.Append(container.NewTabItem("🔓 Decrypt", container.NewScroll(decryptContent)))

	title := widget.NewLabel("Secure Offline Encryption (Argon2id + AES-256-GCM)")
	title.TextStyle = fyne.TextStyle{Bold: true}

	w.SetContent(container.NewBorder(title, nil, nil, nil, tabs))

	w.ShowAndRun()
}
