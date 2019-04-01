# How to add Rich Snippet to Wordpress themes manually using Yoast  + WP-PostRatings 
Here I am to tell you,  how we can easily add rich snippets to our Wordpress theme manually without headache or using complex plugins. 
### Requirements
- You should know about Wordpress & Yoast SEO
- A little knowledge of PHP to understand and make few edits in my codes based on your requirement
- Patience to read this article carefully, you should know that I am lazy as you are and I don't want to be talkative! I'm just writing the important things to make sure you know what we are doing here. 
## Story
As you know, Rich Snippets get considered by Google recently and Yoast only added Schema support for Wordpress 5.0+ new editor and I should tell you that rich snippets have higher priority compare to old meta keywords so your configuration for Yoast meta keywords will be useless in search results if you use another plugin for adding rich snippets in WordPress.

Also, There is a popular plugin called Post Ratings that is very cool and has been downloaded more than 100K from WordPress repository which has built-in support for the rich snippet. That is very cool but as we know, it will override Yoast meta keywords and your search results view won't be the template that you have set in the Yoast appearance section.

it was the story, but what is my solution? 

## My Solution
I thought that I'm able to add rich snippet data manually to WordPress theme easily as I developed the theme my self.
So i started to edit [b] single.php [/b] :

 ### 1. Find the WordPress loop, it should be something like this :
```php
<?php get_header(); ?>
 <div id="content" class="narrowcolumn">

  <?php if (have_posts()) : ?>
   <?php while (have_posts()) : the_post(); ?>
    <!-- START OF LOOP -->
     <div class="post">
     <h2 id="post-<?php the_ID(); ?>">
<a href="<?php the_permalink() ?>" rel="bookmark" title="Permanent Link to <?php the_title_attribute(); ?>"><?php the_title(); ?></a></h2>
     <small><?php the_time('F jS, Y') ?> <!-- by <?php the_author() ?> --></small>
      </div>
      <!-- END OF LOOP -->
    <?php endwhile; ?>
<?php else : ?>
  <h2 class="center">Not Found</h2>
 <p class="center"><?php _e("Sorry, but you are looking for something that isn't here."); ?></p>
  <?php endif; ?>
</div>
<?php get_sidebar(); ?>
<?php get_footer(); ?>
```

### 2. Prepare Your JSON-LD Format, Based on your website
Before you do anything, I should tell you that your content subject and type are really important in the snippet that you want to generate. for example, if your contents are video blog you should use video format or if your website is a blog or you are writing news or article you should use Article/ArticleNews format. So please check out following links to know what format you should use and you can easily prepare and develop your format at the end of this article.

[JSONLD.com (Article) : please check other formats ](https://jsonld.com/article/)
[Google Developer Section ( with more details )](https://developers.google.com/search/docs/data-types/article)


My website contents are articles, so here is what i want to generate : 
```javascript    
 <script type="application/ld+json">
{ 
"@context": "http://schema.org", 
 "@type": "Article",
 "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://google.com/article"
  },
 "headline": "Post Headline",
 "alternativeHeadline": "Meta Description or any other format you want to add",
 "image": "http://cdn.myweb.com/FeaturedImageUrl.jpg",
 "author": {
    "@type": "Person",
    "name": "Soroosh Khodami"
  },
   "publisher": {
    "@type": "Organization",
    "name": "MyWebsite",
    "logo": {
      "@type": "ImageObject",
      "url": "https://cdn.myweb.com/logo.jpg"
    }
    },
 "genre": "My Article Primary Category", 
 "keywords": "My Tags", 
 "wordcount": "Word counts",
 "url": "Permalink of article",
 "datePublished": "PublishDate",
 "dateCreated": "DateCreated",
 "dateModified": "DateModified",
 "description": "Meta Description (I want to use Yoast format)",
 "articleBody": "Whole content of article and its really big.",
 "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "WP-PostRating Average",
    "bestRating": "5",
    "ratingCount": "Count of votes"
  }
 }
</script>
```

### 3.Generate the JSON-LD format
Read the following code carefully, I wrote notes and explanation inside the box. then you could add it anywhere inside the Wordpress loop in single posts, pages or any other post types. 
I prefer to create a new file called 'single_post_rich_snippet.php' and insert the following code into it, then I will include this file anywhere.

```php
<?php

$featured_image = wp_get_attachment_image_src( get_post_thumbnail_id( $post->ID ), 'img_post' ); 
// img_post is name of my thumbnail size ( I've added my own in functions.php ) 
$focuskw = get_post_meta(get_the_ID(), ‘_yoast_wpseo_focuskw’, true);
// It's focus keyword that you enter in Yoast Metabox in post edit page, it's really important
$primary_cat = get_post_meta(get_the_ID(), ‘_yoast_wpseo_primary_category’, true);
// Primary Category is important, for permalink format and meta keywords

if($primary_cat == NULL && $primary_cat == ''){
    $cats = wp_get_post_categories($post->ID);
          $primary_cat = $cats[0];
          // Wordpress posts always have at least one category.
}
$primary_cat = get_cat_name($primary_cat);
// I am building the template for description as the same as i built before in Yoast SEO Titles section, now i am sure that my Rich snippets are the same as my meta tags which is generated by Yoast. 
// My Fromat will be like this : 
// "Wordpress Development (Tutorial),Updated 1 April 2019. For Become professional in wordpress development ..."

$description = $focuskw . " (" .$primary_cat. "), Updated " .get_the_date(). ". " . get_the_excerpt();

// Post keywords
$tags = get_the_tags();
$tags_string = "";
$stripped_content = esc_attr(wp_strip_all_tags( get_the_content() ));                                                            
foreach ( $tags as $tag ) {
    $tags_string .= $tag->name . " ";
}

$post_date = mysql2date( 'c', $post->post_date, false );
$post_modified_date = mysql2date( 'c', $post->post_modified, false);     

?> 
                
<script type="application/ld+json">
{ 
"@context": "http://schema.org", 
 "@type": "Article",
 "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://google.com/article"
  },
 "headline": "<?php the_title(); ?>",
 "alternativeHeadline": "<?php echo $description; ?>",
 "image": "<?php echo $featured_image[0]; ?>",
 "author": {
    "@type": "Person",
    "name": "<?php the_author(); ?>"
  },
   "publisher": {
    "@type": "Organization",
    "name": "MyWebsite",
    "logo": {
      "@type": "ImageObject",
      "url": "https://cdn.myweb.com/logo.jpg"
    }
    },
 "genre": "<?php echo $primary_cat; ?>", 
 "keywords": "<?php echo $tags_string; ?>", 
 "wordcount": " <?php echo str_word_count($stripped_content); ?>",
 "url": "<?php the_permalink() ?>",
 "datePublished": "<?php echo $post_date; ?>",
 "dateCreated": "<?php echo $post_date; ?>",
 "dateModified": "<?php echo $post_modified_date; ?>",
 "description": "<?php echo $description ?>",
 "articleBody": "<?php echo $stripped_content; ?>",
 "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "<?php echo get_post_meta(get_the_ID(),'ratings_average', true); ?>",
    "bestRating": "5",
    "ratingCount": "<?php echo get_post_meta(get_the_ID(), 'ratings_users', true); ?>"
  }
 }
</script>
```
**Note :** if you don't have **WP-PostRatings** or you don't like to show ratings in google search result, just remove "aggreagateRating" key and it's value from end of script. 

**Here is the final result :**
```php
<?php get_header(); ?>
 <div id="content" class="narrowcolumn">

  <?php if (have_posts()) : ?>
   <?php while (have_posts()) : the_post(); ?>
    <!-- START OF LOOP -->
     <div class="post">
     <h2 id="post-<?php the_ID(); ?>">
<a href="<?php the_permalink() ?>" rel="bookmark" title="Permanent Link to <?php the_title_attribute(); ?>"><?php the_title(); ?></a></h2>
     <small><?php the_time('F jS, Y') ?> <!-- by <?php the_author() ?> --></small>
      </div>
      <!-- Rich Snippet -->
      <?php include (TEMPLATEPATH . '/single_post_rich_snippet.php'); ?>
      <!-- END OF LOOP -->
    <?php endwhile; ?>
<?php else : ?>
  <h2 class="center">Not Found</h2>
 <p class="center"><?php _e("Sorry, but you are looking for something that isn't here."); ?></p>
  <?php endif; ?>
</div>
<?php get_sidebar(); ?>
<?php get_footer(); ?>
```

Almost done! we should just validate our codes.

### Validating The Result
For validation of our codes, we could use the Structured Data Testing Tool
https://search.google.com/structured-data/testing-tool

And we could enter our webpage URL and see the result, warnings, and errors. 
if everything looks good, so we should wait in Google Search Console to see the google crawlers results :)

### Yoast Breadcrumb as Rich Snippet
I suggest you use Yoast Breadcrumb in your theme too, it's very easy to add and use to make your search result appearance better. 

### Final Word
Please comment and share your experience with me and help me to make the solution better. 

### Authors and Contributors
- Soroosh Khodami - [Linkedin](https://www.linkedin.com/in/sorooshkhodami/) - [Github](http://github.com/cmandesign/)
