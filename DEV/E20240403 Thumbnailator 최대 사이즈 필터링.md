## 썸네일이 필요했는데..

1. 처음에는 기본 제공 ImageIO API 를 사용해서 resize 를 처리했었다.
``` java
public static byte[] resize(InputStream input, String format, float scale) throws IOException {
    var originalImage = ImageIO.read(input);
    var width = (int) (originalImage.getWidth() * scale);
    var height = (int) (originalImage.getHeight() * scale);
    var newResizedImage = originalImage.getScaledInstance(
        width,
        height,
        Image.SCALE_SMOOTH
    );

    var imageBuff = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
    imageBuff.getGraphics().drawImage(newResizedImage, 0, 0, COLOR, null);
    var buffer = new ByteArrayOutputStream();
    ImageIO.write(imageBuff, format, buffer);
    return buffer.toByteArray();
}
```
2. 리사이즈 과정에서 사진 파일이 회전되는 문제가 있었고, 원본 사진의 Metadata 로 방향을 읽어 회전을 다시 주는 방법이 있다.
```
implementation 'com.drewnoakes:metadata-extractor:2.18.0'
```

3. 방향 확인하는 코드가 관리가 어려울 정도로 더럽고, 하드코딩이 많고, 회전할 때 역시 코드가 많거나 또 다른 라이브러리를 사용해야 편했다.
4. 다시 돌고 돌아 가장 많이 사용한다는 Thumnailator, Exif 를 내부에서 읽어서 리사이클에도 방향을 알아서 맞춰준다.
```
implementation 'net.coobird:thumbnailator:0.4.20'
```
6. 문제는 BufferedImage 에서는 Exif 를 포함하지 않아 방향에 문제가 사라지지 않는다.
![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/60ae895c-227c-4787-8f1a-efd1c2f22c8a)

7. InputStream 을 바로 사용하면 되는데, 그럼 스케일 전의 크기를 모른다. (나는 스케일 전의 크기로 일정 크기 이상이어야 축소 시킨다.)
```
https://github.com/coobird/thumbnailator/issues/11
```
8. 해결은 inputStream 을 사용해서 Exif 정보를 넘기되, filter 를 정의해서 내부에서 BufferedImage 사이즈를 읽어 다시 축소하는 것

``` java
public static byte[] resize(InputStream input, String format, float scale) throws IOException {
    var imageBuff = Thumbnails.of(input)
        .scale(1)
        .useExifOrientation(true)
        .addFilter(getImageFilter(scale))
        .asBufferedImage();
    Graphics graphics = imageBuff.getGraphics();
    graphics.drawImage(imageBuff, 0, 0, COLOR, null);
    ByteArrayOutputStream buffer = new ByteArrayOutputStream();
    ImageIO.write(imageBuff, format, buffer);
    return buffer.toByteArray();
}

private static ImageFilter getImageFilter(float scale) {
    return image -> {
        try {
            if (image.getWidth() * scale > MINIMUM_SIZE && image.getHeight() * scale > MINIMUM_SIZE) {
                return Thumbnails.of(image)
                    .scale(scale)
                    .asBufferedImage();
            }
            return image;
        } catch (IOException e) {
            return image;
        }
    };
}
```

