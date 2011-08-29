require 'rake/clean'
require 'Date'

# no trailing slashes here!
$local_url = "http://localhost/jilles.net"
$production_url = "http://www.jilles.net"

desc 'Development: Rebuild the site'
task :dev => [:dev_config, :js_compile, :build, :tidy_html, :test_broken_links]

desc 'Production: push to github, build and upload'
task :prod => [:push, :prod_config, :js_compile, :build, :tidy_html, :send, :dev_config, :dev]
# finished off in dev_config to ensure prod config doesn't get checked into git
# also, :push needs to be first, otherwise :prod_config changes config files
# which in turn will make the build fail for not having everything checked in

task :default => 'dev'

CLOBBER.include('_flickr.cache')
CLEAN.include('_site/')

desc 'Checks, for dev builds, if there are any internal 404\'s'
task :test_broken_links do
    sh "_bin/wget-test.sh"
end



desc 'Create a new draft post. Usage: rake post title=\'hello, world\''
task :post do
  title = ENV['title']
  slug = "#{Date.today}-#{title.downcase.gsub(/[^\w]+/, '-')}"

  file = File.join(
    File.dirname(__FILE__),
    '_posts',
    slug + '.md'
  )

  categories = `_bin/_list_categories.sh`

  File.open(file, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    published: false
    image_titles: false
    categories:
    #{categories}
    ---

    EOS
  end

  system("vim #{file}")
end


desc 'Configure for development'
task :dev_config do
  puts "* Configuring _config.yml for development... "
  edit_config('prod_build', 'false');
  edit_config('baseurl', $local_url) 
  edit_config('lsi', 'false')
end

desc 'Configure for production'
task :prod_config do
  puts "* Configuring _config.yml for production... "
  edit_config('prod_build', 'true');
  edit_config('baseurl', $production_url) 
  edit_config('lsi', 'true')
end

desc 'Run Jekyll to generate the site'
task :build do
  puts '* Generating static site with Jekyll'
  sh "jekyll"
end

desc 'Upload the _site content'
task :send do
  # perhaps gzip relevant files?
  # find . -type f \( -name "*.xml" -o -name "*.html" -o -name "*.css" -o -name "*.js" \) -exec sh -c "gzip < {} > {}.gz" \;

  # Should test if _site/ exists
  sh "ncftpput -f ~/.ncftp/bookmarks -m -E -S tmp -R / _site/* && echo 'Blog pushed to production'"
  # | growl -H localhost -t 'Jilles.net' || echo 'Blog push FAILED' | growl -H localhost -s -t 'Jilles.net'"
end

task :check_git do
  status = `git status --porcelain --untracked-files=no`
  if status =~ /\S/
    puts " ! Warning: working directory is not clean. Please commit!!"
    puts " ! and don't forget to run 'rake push' after the commit."
    fail
  end
end

desc 'Git push to all remotes'
task :push => :check_git do
  remotes = `git remote`.split
  puts "* Pushing code to all remote repositories (#{remotes.join(", ")})"
  remotes.each do |remote|
    sh "git push #{remote} --all"
  end
end

desc 'Tidy up all html'
task :tidy_html do
    # careful not to touch any content not generated by Jekyll
    sh "echo 'Tidy log' > .tidy_log"
    sh "find ./_site/perma -type f \\( -name '*.xml' -o -name '*.html' \\)    -exec sh -c '(echo {}; tidy -quiet --indent yes -modify -tidy-mark no -wrap 80 {}) &>.tidy_log_tmp; cat .tidy_log_tmp >> .tidy_log' \\;"
    sh "find ./_site -depth 1 -type f \\( -name '*.html' \\)                  -exec sh -c '(echo {}; tidy -quiet --indent yes -modify -tidy-mark no -wrap 80 {}) &>.tidy_log_tmp; cat .tidy_log_tmp >> .tidy_log' \\;"
    sh "rm .tidy_log_tmp"
end

desc 'Closure compile the javascript'
task :js_compile do
    sh "java -jar _bin/closure.jar  --js=js/_titleblock.jquery.js  --js_output_file js/jilles_net.js"
end

## Helpers

def edit_config(option_name, value)
  config = File.read("_config.yml")
  regexp = Regexp.new('(^\s*' + option_name + '\s*:\s*)(\S+)(\s*)$')
  config.sub!(regexp,'\1'+value+'\3')
  File.open("_config.yml", 'w') {|f| f.write(config)}
end

