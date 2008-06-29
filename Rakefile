require 'rubygems'
require 'rake/gempackagetask'
require 'rake/contrib/rubyforgepublisher'

PLUGIN = "ar_openid_store"
NAME = "ar_openid_store"
VERSION = "0.0.1"
AUTHOR = "Dan Webb"
EMAIL = "dan@danwebb.net"
HOMEPAGE = "http://merb-plugins.rubyforge.org/ar_openid_store/"
SUMMARY = "Merb plugin that provides ..."

spec = Gem::Specification.new do |s|
  s.name = NAME
  s.version = VERSION
  s.platform = Gem::Platform::RUBY
  s.has_rdoc = true
  s.extra_rdoc_files = ["README", "LICENSE", 'NOTICE']
  s.summary = SUMMARY
  s.description = s.summary
  s.author = AUTHOR
  s.email = EMAIL
  s.homepage = HOMEPAGE
  s.add_dependency('ruby-openid', '>= 2.0.0')
  s.require_path = 'lib'
  s.autorequire = PLUGIN
  s.files = %w(LICENSE README Rakefile NOTICE) + Dir.glob("{lib}/**/*")
end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.gem_spec = spec
  pkg.need_zip = true
  pkg.need_tar = true
end

task :install => [:package] do
  sh %{sudo gem install pkg/#{NAME}-#{VERSION} --no-update-sources}
end

task :verify_user do
  raise "RUBYFORGE_USER environment variable not set!" unless ENV['RUBYFORGE_USER']
end

desc "Publish gem+tgz+zip on RubyForge. You must make sure lib/version.rb is aligned with the CHANGELOG file"
task :publish_packages => [:verify_user, :package] do
  package_name = [spec.name, spec.version].join '-'
  
  release_files = FileList[
    "pkg/#{package_name}.gem",
    "pkg/#{package_name}.tgz",
    "pkg/#{package_name}.zip"
  ]
  unless spec.version =~ /RC[0-9]$/
    require 'meta_project'
    require 'rake/contrib/xforge'

    Rake::XForge::Release.new(MetaProject::Project::XForge::RubyForge.new('merbopenid')) do |xf|
      # Never hardcode user name and password in the Rakefile!
      xf.user_name = ENV['RUBYFORGE_USER']
      xf.files = release_files.to_a
      xf.release_name = "ActiveRecord OpenID Store #{spec.version}"
      xf.release_notes = ''
      xf.release_changes = ''
    end
  else
    puts "SINCE THIS IS A PRERELEASE, FILES ARE UPLOADED WITH SSH, NOT TO THE RUBYFORGE FILE SECTION"
    puts "YOU MUST TYPE THE PASSWORD #{release_files.length} TIMES..."

    host = "merbopenid-website@rubyforge.org"
    remote_dir = "/var/www/gforge-projects/merbopenid"

    publisher = Rake::SshFilePublisher.new(
      host,
      remote_dir,
      File.dirname(__FILE__),
      *release_files
    )
    publisher.upload

    puts "UPLOADED THE FOLLOWING FILES:"
    release_files.each do |file|
      name = file.match(/pkg\/(.*)/)[1]
      puts "* http://merbopenid.rubyforge.org/#{name}"
    end

    puts "They are not linked to anywhere, so don't forget to tell people!"
  end
end

namespace :jruby do

  desc "Run :package and install the resulting .gem with jruby"
  task :install => :package do
    sh %{#{SUDO} jruby -S gem install pkg/#{NAME}-#{Merb::VERSION}.gem --no-rdoc --no-ri}
  end
  
end