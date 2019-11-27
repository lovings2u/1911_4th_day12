# Day12

- 1:N
  - 댓글 관계
  
  - 모델에 메소드 달기
  
    ```python
    
    class Article(models.Model):
        contents = models.TextField()
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
        # 원본 이미지를 저장해두고
        # image = models.ImageField(blank=True)
        
        # 수정된 이미지도 따로 저장해서 사용
        # image_resized = ProcessedImageField(
        #     # source='image',
        #     upload_to='articles/images',
        #     processors=[ResizeToFill(200,200)],
        #     format='JPEG',
        #     options={'quality': 90}
        # )
    
        # 이미지의 썸네일을 생성해줌
        # media/CACHE에 원본 이미지의 썸네일을 자동생성
        # image_thumbnail = ImageSpecField(
        #     source='image',
        #     processors=[Thumbnail(200,200)],
        #     format='JPEG',
        #     options={'quality': 90}
        # )
        def comments(self):
            return Comment.objects.filter(article_id=self.id)
        def article_images(self):
            return ArticleImages.objects.filter(article_id=self.id)
    
    class ArticleImages(models.Model):
        article = models.ForeignKey(Article, on_delete=models.CASCADE)
        image = models.ImageField(blank=True)
        image_thumbnail = ImageSpecField(
            source='image',
            processors=[Thumbnail(300,300)],
            format='JPEG',
            options={'quality': 90}
        )
    
    class Comment(models.Model):
        contents = models.TextField()
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
        article = models.ForeignKey(Article, on_delete=models.CASCADE)
    
    
    ```
  
- 댓글 수정, 삭제

  ```python
  # article/views.py
  def index(request):
      if request.method == "POST":
          article = Article()
          article.contents = request.POST["contents"]
          # 원본 이미지를 저장
          # article.image = request.FILES["image"]
          # 원본 이미지를 프로세싱 한 이미지를 저장
          # article.image_resized = request.FILES["image"]
          article.save()
          for image in request.FILES.getlist("image"):
              ArticleImages.objects.create(article_id=article.id, image=image)
          return redirect('articles')
      else:
          articles = Article.objects.all().order_by("created_at").reverse()
          context = {
              'articles': articles
          }
          return render(request, 'index.html', context)
  
  def edit(request, article_id):
      article = Article.objects.get(id=article_id)
      if request.method == "POST":
          article.contents = request.POST["contents"]
          article.save()
          return redirect('articles')
      else:
          context = {
              'article': article
          }
          return render(request, 'article/edit.html', context)
  
  def delete(request, article_id):
      article = Article.objects.get(id=article_id)
      article.delete()
      return redirect('articles')
  
  ```

  - 이미지 수정 관련해서는 따로 설명하지 않음

  ```html
  <!-- bast.html -->
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>Document</title>
  
      <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
      {% block stylesheet %}
      {% endblock %}
      <script src="https://kit.fontawesome.com/faa32ab184.js" crossorigin="anonymous"></script>
  </head>
  <body>
      <div class="d-flex flex-column flex-md-row align-items-center p-3 px-md-4 mb-3 bg-white border-bottom shadow-sm">
          <h5 class="my-0 mr-md-auto font-weight-normal">INSTAGRAM</h5>
          <a class="btn btn-outline-primary" href="#">Sign up</a>
      </div>
      {% block content %}
      {% endblock %}
  
      <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
      <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
  
      {% block script %}
      {% endblock %}
      
  </body>
  </html>
  ```

  ```html
  <!-- index.html -->
  
  {% block stylesheet %}
  <style>
  .container {
      padding-right: 10em !important;
      padding-left: 10em !important;
  }
  </style>
  {% endblock %}
  
  {% block content %}
  <div class="container">
      <div class="card">
          <form action="{% url 'articles' %}" method="POST" enctype="multipart/form-data">
              <input type="hidden" name="csrfmiddlewaretoken" value="{{csrf_token}}">
              <div class="card-body">
                  <textarea name="contents" class="form-control" rows="5"></textarea>
                  <div class="input-group mt-3">
                      <div class="input-group-prepend">
                          <span class="input-group-text" id="inputGroupFileAddon01">Upload</span>
                      </div>
                      <div class="custom-file">
                          <input multiple name="image" type="file" class="custom-file-input" id="inputGroupFile01" aria-describedby="inputGroupFileAddon01">
                          <label class="custom-file-label" for="inputGroupFile01">Choose file</label>
                      </div>
                  </div>
              </div>
              <div class="card-footer text-right">
                  <input type="submit" class="btn btn-success" value="작성하기">
              </div>
          </form>
      </div>
  </div>
  
  <div class="container">
      {% for article in articles %}
      <div class="row mt-4">
          <div class="card col-12">
              {% if article.article_images %}
              <div id="carouselExampleControls" class="carousel slide" data-ride="carousel">
                  <div class="carousel-inner">
                      {% for image in article.article_images %}
                      <div class="carousel-item {%if image.image_thumbnail.url == article.article_images.first.image_thumbnail.url%}active{% endif %}">
                          <img src="{{image.image_thumbnail.url}}" class="d-block w-100" alt="...">
                      </div>
                      {% endfor %}
                  </div>
                  <a class="carousel-control-prev" href="#carouselExampleControls" role="button" data-slide="prev">
                      <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                      <span class="sr-only">Previous</span>
                  </a>
                  <a class="carousel-control-next" href="#carouselExampleControls" role="button" data-slide="next">
                      <span class="carousel-control-next-icon" aria-hidden="true"></span>
                      <span class="sr-only">Next</span>
                  </a>
              </div>
              {% endif %}
              <div class="card-body">
                  <div class="article" style="min-height: 9rem;">
                      <p class="card-text">{{article.contents}}</p>
                  </div>
                  <div class="text-center">
                      <a href="{% url 'edit' article.id %}" class="btn btn-warning text-white"><i class="fas fa-edit"></i></a>
                      <a href="{% url 'delete' article.id %}" class="btn btn-danger text-white"><i class="fas fa-trash-alt"></i></a>
                  </div>
              </div>
              <ul class="list-group list-group-flush">
                  <li class="list-group-item">
                      <form action="{% url 'comments' %}" method="POST">
                          <input type="hidden" name="csrfmiddlewaretoken" value="{{csrf_token}}">
                          <input type="hidden" name="article_id" value="{{article.id}}">
                          <div class="row">
                              <div class="col-9">
                                  <input type="text" class="form-control" name="contents">
                              </div>
                              <div class="col-3 text-center">
                                  <input type="submit" class="btn btn-primary" value="쓰기">
                              </div>
                          </div>
                      </form>
                  </li>
                  {% for comment in article.comments %}
                  <li class="list-group-item">
                      <i class="fas fa-comment-dots"></i> 
                      {{comment.contents}}
                      <span class="float-right">
                          <a href="{% url 'edit_comment' comment.id %}" class="btn btn-warning text-white"><i class="fas fa-edit"></i></a>
                          <a href="{% url 'delete_comment' comment.id %}" class="btn btn-danger text-white"><i class="fas fa-trash-alt"></i></a>
                      </span>
                  </li>
                  {% endfor %}
              </ul>
          </div>
      </div>
      {% endfor %}
  </div>
  
    
  {% endblock %}
  
  ```

  - JS를 이용하지 않고 기본적인 form을 통해 사용자가 만들고자 하는 article 객체를 생성

- 이미지 업로드 시(한장)

- `pip install pillow`

- `pip install django-imagekit`

- *만약 `pip install imagekit`을 함께 했다면 이미지 킷은 지워준다.*

  - *models.py*

    ```python
    ...
    image = models.ImageField(blank=True)
        image_thumbnail = ImageSpecField(
            source='image',
            processors=[Thumbnail(300,300)],
            format='JPEG',
            options={'quality': 90}
        )
    ...
    ```

  - *settings.py*

    ```python
    ...
    INSTALLED_APPS = [
        'article',
        'imagekit',
        ...
    ]
    ...
    
    STATIC_URL = '/static/'
    
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    MEDIA_URL = '/media/'
    ```

    - MEDIA_URL은 업로드 한 이미지가 저장될 장소

  - *views.py*

    ```python
    ...
    article = Article()
    article.contents = request.POST["contents"]
    article.image = request.FILES["image"]
    ```

    - 다음을 통해 이미지를 저장
    - 리사이즈 된 이미지를 사용할 경우 원본이 이미지와 함께 자동으로 버저닝되며, 썸네일 이용시에는 원본 이미지만 저장되고 나머지는 CACHE에 남아 있다.
