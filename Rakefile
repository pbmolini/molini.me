task :default => :preview

#
# Set the values of
#
$deploy_url = "http://molini.me"    # where the system will live
# $deploy_dir = "user@host:~/some-location/"        # where the sources live
# $post_dir   = "_posts/"                           # where posts are created
#
# ... or load them from a file, e.g.:
#
# load '_rake-configuration.rb'

#
# Tasks start here
#

desc 'Clean up generated site'
task :clean do
  cleanup
end

desc 'Preview on local machine (server with --auto)'
task :preview => :clean do
  set_url('http://localhost:4000')
  sh('compass watch &')
  jekyll('serve --watch')
end

desc 'Static build (build using filesystem)'
task :build_static => :clean do
  set_url(Dir.getwd + "/_site")
  jekyll('build')
end

desc 'Checks whether ruby-git is installed'
task :rubygit do
  begin
    require 'git'
  rescue LoadError
    puts 'The git gem is missing! Installing...'
    `gem install git`
    abort 'Done! Now run rake deploy again'
  end
end

desc 'Build for deployment (but do not deploy)'
task :build => [:rubygit,:clean] do
  git_push
  jekyll('build')
end

desc 'Build and deploy to remote server'
task :deploy => :build do
#  sh "rsync -avz --delete _site/ #{$deploy_dir}"
#  File.open("_last_deploy.txt", 'w') {|f| f.write(Time.new) }
end

desc 'Create a post'
task :create_post, [:post, :date, :content] do |t, args|
  if args.post == nil or
    (args.date and args.date.match(/[0-9]+-[0-9]+-[0-9]+/) == nil) then
    puts "Usage: create post TITLE [DATE]"
    puts "Date is in the form: Y-m-d"
    exit 1
  end

  post_title= args.post
  post_date= args.date || Time.new.strftime("%Y-%m-%d %H:%M:%S")

	# remove the time from post_date (the filename does not support it)
	filename = post_date[0..9] + "-" + post_title.gsub(' ', '_') + ".textile"

	# generate a unique filename appending a number
	i = 1
	while File.exists?($post_dir + filename) do
	  filename = post_date[0..9] + "-" + 
	  post_title.gsub(' ', '_') + "-" + i.to_s +
	  ".textile"
	  i += 1
	end

	# the if is not really necessary anymore
	if not File.exists?($post_dir + filename) then
	  File.open($post_dir + filename, 'w') do |f|
	    f.puts "---"
	    f.puts "title: \"#{post_title}\""
	    f.puts "layout: default"
	    f.puts "date: #{post_date}"
	    f.puts "---"
	    f.puts args.content if args.content != nil
	  end

	  puts "Post created under \"#{$post_dir}#{filename}\""

	  sh "open \"#{$post_dir}#{filename}\"" if args.content == nil
	else
	  puts "A post with the same name already exists. Aborted."
	end
	# puts "You might want to: edit #{$post_dir}#{filename}"
end

desc 'Check links for site already running on localhost:4000'
task :check_links do
  begin
    require 'anemone'
    root = 'http://localhost:4000/'
    Anemone.crawl(root, :discard_page_bodies => true) do |anemone|
      anemone.after_crawl do |pagestore|
        broken_links = Hash.new { |h, k| h[k] = [] }
        pagestore.each_value do |page|
          if page.code != 200
            referrers = pagestore.pages_linking_to(page.url)
            referrers.each do |referrer|
              broken_links[referrer] << page
            end
          end
        end
        broken_links.each do |referrer, pages|
          puts "#{referrer.url} contains the following broken links:"
          pages.each do |page|
            puts "  HTTP #{page.code} #{page.url}"
          end
        end
      end
    end

  rescue LoadError
    abort 'Install anemone gem: gem install anemone'
  end
end

def cleanup
  sh 'rm -rf _site'
end

def jekyll(directives = '')
  sh 'jekyll ' + directives
end

# set the url in the configuration file
def set_url(url)
  config_filename = "_config.yml"

  text = File.read(config_filename)
  url_directive = Regexp.new(/url: .*$/)
  if text.match(url_directive)
    puts = text.gsub(url_directive, "url: #{url}")
  else
    puts = text + "\nurl: #{url}"
  end
  File.open(config_filename, "w") { |file| file << puts }
end

def git_push
  require 'git'

  set_url($deploy_url)
  git = Git.open(Dir.getwd)

  git_status = git.status

  if git_status.untracked.any?
    puts "You have untracked files! Please add them manually using 'git add' before deploying!"
  elsif git_status.changed.any?
    puts "The following files have been changed and must be committed"
    git_status.changed.each do |filename, file|
      puts "  * #{filename}"
    end
    puts
    puts "Please enter a commit message: "
    commit_message = STDIN.gets
    git.commit_all(commit_message)
  end

  git.push('origin', 'gh-pages')
end