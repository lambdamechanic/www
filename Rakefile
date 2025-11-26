require "fileutils"
require "date"
require "tmpdir"

POSTS_DIR = File.expand_path("_posts", __dir__)

def slugify(title)
  title.downcase
       .gsub(/[^a-z0-9]+/, "-")
       .gsub(/-{2,}/, "-")
       .gsub(/^-|-$|\A\z/, "")
end

def ensure_posts_dir!
  FileUtils.mkdir_p(POSTS_DIR)
end

desc "Create a new blog post via: rake new_post title='Title' [date=YYYY-MM-DD] [slug=custom-slug]"
task :new_post, [:title, :date, :slug] do |_t, args|
  raw_title = args[:title] || ENV["title"] || ENV["TITLE"]
  title = raw_title&.strip
  abort "Please provide a title via title='My Post Title'" if title.nil? || title.empty?

  date_input = args[:date] || ENV["date"] || ENV["DATE"] || Date.today.to_s
  date = Date.parse(date_input)

  raw_slug = args[:slug] || ENV["slug"] || ENV["SLUG"]
  slug = raw_slug&.strip
  slug = slugify(slug) unless slug.nil? || slug.empty?
  slug ||= slugify(title)
  abort "Could not derive a slug from '#{title}'." if slug.nil? || slug.empty?

  ensure_posts_dir!
  filename = File.join(POSTS_DIR, format("%<date>s-%<slug>s.md", date: date.strftime("%Y-%m-%d"), slug: slug))

  abort "Post already exists: #{filename}" if File.exist?(filename)

  front_matter = <<~POST
    ---
    title: #{title}
    description: TODO: add a sharp, single-sentence summary.
    author: Mark Wotton
    ---

    Start with a short lede that states the problem.
    Keep each sentence on its own line to follow Sembr formatting.
  POST

  File.write(filename, front_matter)

  puts "Created #{filename}"
end

desc "Build the site and fail if any internal links are broken"
task :check_links do
  Dir.mktmpdir("jekyll-build-") do |dir|
    sh "bundle exec jekyll build --destination #{dir}"
    sh "bundle exec htmlproofer #{dir} --disable-external"
  end
end
