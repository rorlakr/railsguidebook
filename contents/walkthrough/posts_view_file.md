# posts 뷰 변경

`index` 액션의 뷰 파일 중 해당 부분을 아래와 같이 변경하고, 인스턴스 변수(`@posts`)를 사용하는 부분만 집중해서 보자.

```html
<% @posts.each do |post| %>
  <tr>
    <td><%= post.title %></td>
    <td>
      <%= link_to 'Show', [post.bulletin, post], class:'btn btn-default' %>
      <%= link_to 'Edit', edit_bulletin_post_path(post.bulletin, post), class:'btn btn-default' %>
      <%= link_to 'Destroy', [post.bulletin, post], method: :delete, data: { confirm: 'Are you sure?' }, class:'btn btn-default' %>
    </td>
  </tr>
<% end %>
```

`@posts` 인스턴스 변수는 배열을 가진다. 루비의 배열 메소드인 `each`는 리시버(`.each` 앞에 있는 객체)의 각 요소를 하나씩 반복해서 `do` 블록 변수(여기서는 post)로 넘겨 준다. 따라서 `<%= post.title %>`는 `post` 객체의 `title` 속성값이 삽입된다.


### 기타 posts 뷰 파일의 변경 사항

* `app/views/posts/_form.html.erb`에서 아래와 같이 수정한다.

  ```
-<%= simple_form_for(@post) do |f| %>
+<%= simple_form_for([@bulletin, @post]) do |f| %>
```


* `app/views/posts/edit.html.erb`

  ```
-<%= link_to 'Show', @post, class: 'btn btn-default' %>
-<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
+<%= link_to 'Show', [@post.bulletin, @post], class: 'btn btn-default' %>
+<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```
* `app/views/posts/index.html.erb`

  ```
-<%= link_to 'New Post', post_path, class: 'btn btn-default' %>
+<%= link_to 'New Post', new_bulletin_post_path, class: 'btn btn-default' %>
  ```

* `app/views/posts/new.html.erb`

  ```
-<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
+<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```


* `app/views/posts/show.html.erb`

  ```
-<%= link_to 'Edit', edit_post_path(@post), class: 'btn btn-default' %>
-<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
+<%= link_to 'Edit', edit_bulletin_post_path(@post.bulletin, @post), class: 'btn btn-default' %>
+<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```

### welcome#index 뷰 파일 변경

`welcome` 컨트롤러의 `index` 액션 뷰 파일(`app/views/welcome/index.html.erb')을 열고 `posts_path`를 bulletin_posts_path(1)`로 변경한다. 이것은 `공지사항` 게시판으로 이동하기 위한 것이다.

```html
<%= link_to "글작성", bulletin_posts_path(1), class:'btn btn-default' %>
```

### 브랜드 로고 링크 업데이트

애플리케이션 레이아웃 파일을 열고 21번째 코드라인에서,

```erb
<a class="navbar-brand" href="#">Rails<i>Cafe</i></a>
```

`href` 속성값을 `root` 경로로 지정한다. 이 때는 `link_to` 헬퍼 메소드를 이용하여 지정해 보도록 한다.

```erb
<%= link_to raw("Rails<i>Cafe</i>"), root_path, class:'navbar-brand' %>
```

위에서 사용한 [`raw()`](http://api.rubyonrails.org/classes/ActionView/Helpers/OutputSafetyHelper.html#method-i-raw) 헬퍼메소드는 문자열 내의 특수문자(여기서는 html 태그)를 이스케이핑하지 않은 채로 출력해 준다. 레일스에서는 디폴트 상태에서 모든 문자열을 이스케이핑하는데, 여기서는 문자열내의 `<i></i>` html 태그가 동작하도록 할 필요가 있기 때문에 `raw()` 메소들 사용하였다. 그러나, 이 메소드는 악의적 해커들의 공격여지를 줄 수 있기 때문에, 사용자의 입력 데이터에 바로 적용해서는 안된다는 것을 기억해 두자. 

### navbar 메뉴변경

`app/views/layouts/application.html.erb` 파일에서 아래와 같이 `<ul class='nav navbar-nav'>` 부분을 아래와 같이 변경한다.

```html
<ul class="nav navbar-nav">
  <li class="<%= params[:bulletin_id] == '1' ? 'active' : '' %>"><%= link_to '공지사항', bulletin_posts_path('1') %></li>
  <li class="<%= params[:bulletin_id] == '2' ? 'active' : '' %>"><%= link_to '새소식', bulletin_posts_path('2') %></li>
  <li class="<%= params[:bulletin_id] == '3' ? 'active' : '' %>"><%= link_to '가입인사', bulletin_posts_path('3') %></li>
</ul>
```

그리고, `http://localhost:3000/bulletins` 로 접속한 후 `New Bulletin` 버튼을 클릭하여 "새소식"과 "가입인사" 게시판을 추가한다. 

| bulletin_id | 값  |
|:--------:|--------|
| 1 | 공지사항 |
| 2 | 새소식 |
| 3 | 가입인사 |

우리가 의도한 바는 상단 메뉴 항목를 클릭하면 해당 게시판으로 이동하고 해당 항목이 주황색의 글씨로 표시되도록 하는 것이다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_22-02-30_zpsb40d5eb8.png)

### 게시판 ID 찾아내서 할당하기

생성 순서에 따라 게시판 ID는 달라질 수 있다. 레일스가 레코드를 생성할 때 ID를 자동으로 할당하기 때문이다. DB에 직접 접속해서 알맞은 게시판 ID(`bulletin_id`) 값을 알아내서 할당해야 한다. 레일즈 서버를 잠시 중단하고 DB에 접속한다.

``` bash
$ bin/rails db
SQLite version 3.8.2 2013-12-06 14:53:30
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite>
```

`.tables`를 입력하면 지금까지 생성한 `bulletins` 테이블과 `posts` 테이블을 확인할 수 있다.

``` bash
sqlite> .tables
bulletins          posts              schema_migrations
```

`bulletins` 테이블에 있는 모든 자료를 `select` 문으로 확인해보자. 앞서 웹 브라우저에서 생성한 공지사항 게시판의 첫 필드 16이 게시판 ID, `bulletin_id` 값을 의미한다.

``` bash
sqlite> select * from bulletins;
16|공지사항|공지사항을 올리는 게시판이에요|2015-02-05 02:13:16.303096|2015-02-05 02:13:16.303096|bulletin
```

이렇게 되면 공지사항 게시판으로 이동하기 위한 `link_to` 메소드에서 `bulletin_id`는 1이 아닌 16으로 변경해야 한다. 공지사항 뿐만아니라 새소식, 가입인사 게시판 모두 마찬가지이다.

``` html
<ul class="nav navbar-nav">
  <li class="<%= params[:bulletin_id] == '16' ? 'active' : '' %>"><%= link_to '공지사항', bulletin_posts_path('16') %></li>
  ...
```

잊지 말고 `welcome` 컨트롤러의 `index` 액션 뷰 파일도 `bulletin_id`를 수정하자.

```html
<%= link_to "글작성", bulletin_posts_path(16), class:'btn btn-default' %>
```

---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_09
