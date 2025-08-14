# Android CI/CD Setup Guide

## 📋 Overview

This repository includes a fully optimized GitHub Actions workflow for building and releasing Android APKs. The workflow includes:

✅ **Unit Testing** - Ensures code passes tests before building APK  
✅ **gradle/actions@v3** - Official Gradle action with wrapper management and caching  
✅ **Direct Google Play Upload** - Automatically uploads signed Release APKs to Google Play Store  
✅ **Android SDK Caching** - Reduces SDK download time on subsequent runs  
✅ **Gradle Caching** - Speeds up dependency downloads  
✅ **All SDK License Support** - Accepts all licenses including Google-TV and XR  
✅ **Release Signing Support** - Optional support for signing release builds  

## 🚀 Quick Start

1. **Save the workflow file**: The workflow is already saved at `.github/workflows/build-apk.yml`
2. **Set up secrets** (see below)
3. **Push changes** to trigger the workflow

## 🔐 Required Secrets

Configure these secrets in your repository: `Settings ➜ Secrets and variables ➜ Actions ➜ New repository secret`

### For Release APK Signing (Optional)

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `KEYSTORE_BASE64` | Your keystore file (`.jks`) encoded in Base64 | `MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQC...` |
| `KEYSTORE_PASSWORD` | Password for the keystore | `my_keystore_password` |
| `KEY_ALIAS` | Alias name inside the keystore | `my_app_key` |
| `KEY_PASSWORD` | Password for the key inside the keystore | `my_key_password` |

### For Google Play Store Upload (Optional)

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` | Service account JSON content (as text) | `{"type": "service_account", "project_id": "..."}` |
| `PLAY_PACKAGE_NAME` | Your app's package name | `com.example.myapp` |
| `PLAY_TRACK` | Play Store track (optional, defaults to `internal`) | `internal`, `alpha`, `beta`, or `production` |

## 🔧 How to Generate Required Files

### 1. Create Keystore for Release Signing

```bash
keytool -genkey -v -keystore my-app-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my_app_key
```

### 2. Convert Keystore to Base64

```bash
base64 -i my-app-key.jks | tr -d '\n'
```

Copy the output and paste it as the `KEYSTORE_BASE64` secret.

### 3. Create Google Play Service Account

1. Go to [Google Play Console](https://play.google.com/console)
2. Navigate to **Setup ➜ API access**
3. Create or link a Google Cloud project
4. Create a service account with **Release Manager** permissions
5. Download the JSON key file
6. Copy the entire JSON content and paste it as `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` secret

## 🏗️ Workflow Features

### What the Workflow Does

1. **Environment Setup** (Ubuntu 24.04, JDK 17)
2. **SDK Management** with intelligent caching
3. **License Acceptance** for all Android SDK components
4. **Unit Testing** with failure detection
5. **Debug APK Build** (always runs)
6. **Release APK Build** (if keystore secrets provided)
7. **Artifact Upload** (downloadable from GitHub Actions)
8. **Google Play Upload** (if Play Store secrets provided)

### Caching Strategy

- **Android SDK Cache**: Reduces download time from ~5 minutes to ~30 seconds
- **Gradle Cache**: Speeds up dependency resolution
- **Cache Size**: ~2-3 GB total (within GitHub's 5 GB limit per repository)

### Build Outputs

After each successful run, you can download:

- **Debug APK**: Always available in Actions artifacts
- **Release APK**: Available if keystore secrets are configured
- **Google Play**: Automatically uploaded to specified track

## ⚙️ Customization Options

### Change Android SDK Version

Edit the `android-sdk-manifest.txt` creation step in the workflow:

```yaml
- name: Create SDK manifest file (cache key helper)
  run: |
    cat <<EOF > android-sdk-manifest.txt
    platform-tools
    platforms;android-35  # Change this
    build-tools;35.0.0    # And this
    EOF
```

### Modify Google Play Track

Set the `PLAY_TRACK` secret to one of:
- `internal` (default)
- `alpha`
- `beta` 
- `production`

### Add Additional Gradle Tasks

Modify the build steps to include additional tasks:

```yaml
- name: Build and Test
  uses: gradle/actions@v3
  with:
    arguments: clean test assembleDebug lint --no-daemon
```

## 🔍 Troubleshooting

### Common Issues

1. **SDK License Errors**: The workflow includes comprehensive license acceptance
2. **Build Failures**: Check the unit tests step - builds stop if tests fail
3. **Cache Issues**: Clear cache by updating the SDK manifest file
4. **Keystore Errors**: Verify Base64 encoding and secret values

### Debugging Steps

1. Check the **Actions** tab for detailed logs
2. Verify all required secrets are set correctly
3. Ensure your Android project has proper Gradle configuration
4. Test keystore and passwords locally before setting secrets

### Performance Optimization

- **Large Projects**: Consider splitting into multiple jobs
- **Cache Size**: Monitor cache usage in repository settings
- **Parallel Builds**: Use Gradle parallel execution for multi-module projects

## 📁 Project Structure

Your Android project should have the standard structure:

```
your-repo/
├── .github/
│   └── workflows/
│       └── build-apk.yml
├── app/
│   ├── build.gradle(.kts)
│   └── src/
├── gradle/
│   └── wrapper/
├── build.gradle(.kts)
└── settings.gradle(.kts)
```

## 🎯 Next Steps

1. **Configure Secrets**: Set up the required secrets for your use case
2. **Test Workflow**: Push a commit to trigger the workflow
3. **Monitor First Run**: Check logs for any configuration issues
4. **Download APKs**: Get your built APKs from the Actions artifacts
5. **Verify Upload**: Check Google Play Console for uploaded releases

## 🆘 Support

If you encounter issues:

1. Check the workflow logs in the Actions tab
2. Verify your secrets are correctly configured
3. Ensure your Android project follows standard Gradle conventions
4. Test keystore and Google Play credentials independently

---

🎉 **You're all set!** The workflow will now automatically build, test, and release your Android app on every push to the main branch.