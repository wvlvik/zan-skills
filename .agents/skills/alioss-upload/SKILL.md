---
name: alioss-upload
description: Upload files and directories to Alibaba Cloud OSS using internal oss-up binary (AK/SK baked in). Internal use only — do NOT release publicly. Use when users request "upload to OSS", "阿里云OSS上传", "upload images", "批量上传图片", or any file upload task involving Alibaba Cloud OSS.
---

# OSS Upload (内部专用)

> ⚠️ **安全提醒**
>
> `bin/oss-up` 二进制含编译进去的 AK/SK。**严禁**：
> - 提交公开仓库
> - 发外部群/邮件
> - 拷贝到不可信机器
>
> 仅公司内部使用。

## 安装

```bash
mkdir -p ~/.claude/skills
tar xzf alioss-upload-skill.tar.gz -C ~/.claude/skills/
chmod +x ~/.claude/skills/alioss-upload/bin/oss-up
xattr -c ~/.claude/skills/alioss-upload/bin/oss-up

# mac Apple Silicon 需要 (装过可跳)
softwareupdate --install-rosetta --agree-to-license
```

## 验证

```bash
~/.claude/skills/alioss-upload/bin/oss-up --help
# 应输出 "oss-up — Alibaba Cloud OSS upload CLI"
```

## Binary Location

```
~/.claude/skills/alioss-upload/bin/oss-up
```

No environment variables needed — credentials are baked into the binary.

## Quick Start

```bash
OSS_UP=~/.claude/skills/alioss-upload/bin/oss-up

# Upload single file
$OSS_UP file.txt

# Upload single image (public-read, default prefix: statics/i/img)
$OSS_UP --image photo.jpg

# Upload multiple images
$OSS_UP --images img1.jpg img2.png photo.jpg

# Upload to custom prefix
$OSS_UP --images photo1.jpg photo2.jpg --prefix assets/img

# Upload directory
$OSS_UP --dir ./dist --prefix releases/v1.0

# Upload large file (resumable multipart)
$OSS_UP --large video.mp4
```

## Output Behavior

After upload succeeds, display URL directly to user:

```markdown
✓ Upload complete!
  URL: https://bucket.oss-cn-hangzhou.aliyuncs.com/path/to/file
  Key: path/to/file
  Size: 1.5 MB
```

For `--image` / `--images`, URL is publicly accessible.

## Specify File Paths

When user wants to upload files, accept paths in these forms:

**1. Direct path (推荐)**
```
用户: 上传 /Users/me/Desktop/photo.png 到 OSS
```

**2. Multiple files**
```
用户: 上传这些文件: /path/a.png, /path/b.jpg
```

**3. Current file in editor** — ask for full path:
```
助手: 请提供文件的完整路径，例如: /Users/me/Desktop/image.png
```

**Note**: This model does not support image input directly. User must provide file paths as text.

## Upload Modes

| Flag | Use Case |
|------|----------|
| *(none)* | Single file, private ACL |
| `--image` | Single image, public-read, prefix `statics/i/img` |
| `--images` | Multiple images, auto-sanitize filenames |
| `--dir` | Directory contents |
| `--large` | Resumable multipart for large files |

## Key Options

```
--prefix, -p     OSS key prefix (default for images: statics/i/img)
--key, -k        OSS object key (single file)
--domain, -D     Custom domain for display URLs (e.g., cdn.example.com)
--acl            private | public-read | public-read-write
--storage-class  Standard | IA | Archive
--pattern        File glob for --dir (default: *)
--no-recursive   Skip subdirectories in --dir
--sanitize, -s   Rename Chinese/special-char filenames to MD5 hash
--part-size      Part size in MB for --large (default: 6)
--parallel       Parallel uploads for --large (default: 3)
--quiet, -q      Suppress progress output
--json           Output raw JSON response
```

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| `NoSuchBucket` | Bucket doesn't exist | Create in OSS console |
| `AccessDenied` | Permission denied | Contact admin |
| `EntityTooLarge` | File > 5GB | Use `--large` |
| binary not found | Not installed | Run install steps above |
| permission denied | Not executable | `chmod +x bin/oss-up` |
| "Bad CPU type" | Wrong arch binary | Install Rosetta (Apple Silicon) |

## OSS URL Format

```
https://{bucket}.oss-{region}.aliyuncs.com/{key}
```

With custom domain:
```
https://cdn.example.com/{key}
```

## 自然语言触发示例

- "上传 ~/Desktop/photo.png 到 OSS"
- "批量上传这些图片"
- "把 ./dist 上传到 OSS releases/v1.0"
- "upload this screenshot to OSS"
