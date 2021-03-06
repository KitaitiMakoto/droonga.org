# -*- ruby -*-
#
# Copyright (C) 2013-2014 Droonga Project
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License version 2.1 as published by the Free Software Foundation.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA

require "bundler/setup"

require "pathname"
require "yaml"

require "jekyll/task/i18n"

def env_value(name)
  value = ENV[name]
  raise "Specify #{name} environment variable" if value.nil?
  value
end

task :default => "jekyll:i18n:translate"

Jekyll::Task::I18n.define do |task|
  task.locales = ["ja"]
  task.translator_name = "Droonga Project"
  task.translator_email = "droonga@groonga.org"
  task.files = Rake::FileList["**/*.md"]
  task.files += Rake::FileList["{reference,tutorial}/**/*.html"]
  task.files -= Rake::FileList["README.md"]
  task.files -= Rake::FileList["_*/**/*.md"]
  task.files -= Rake::FileList["news/**/*.md"]
  task.files -= Rake::FileList["vendor/**/*.*"]
  task.locales.each do |locale|
    task.files -= Rake::FileList["#{locale}/**/*.md"]
  end
  task.custom_translator = lambda do |original, translated, path|
    notice = <<-NOTICE
{% comment %}
##############################################
  THIS FILE IS AUTOMATICALLY GENERATED FROM
  "#{path.po_file}"
  DO NOT EDIT THIS FILE MANUALLY!
##############################################
{% endcomment %}
    NOTICE
    if /^---+$/ =~ translated
      translated = translated.split(/^---+\n/)
      translated[2] = "\n#{notice}\n#{translated[2]}"
      translated.join("---\n")
    else
      "\n#{notice}\n#{translated}"
    end
  end
end

desc "Release a new version"
task :release do
  new_version = env_value("NEW_VERSION")
  config_yaml_path = Pathname.new("_config.yml")
  config_yaml_data = config_yaml_path.read
  config = YAML.load(config_yaml_data)
  current_version = config["droonga_version"]

  targets = []
  targets.concat(Pathname.glob("_po/*/{tutorial,reference}/"))
  targets.concat(Pathname.glob("{tutorial,reference}/"))
  targets.each do |target|
    new_dir = target + new_version
    current_dir = target + current_version
    next unless current_dir.exist?
    rm_rf(new_dir)
    cp_r(current_dir, new_dir)
  end

  new_config = config.merge("droonga_version" => new_version)
  config_yaml_path.open("w") do |config_yaml|
    config_yaml.print(new_config.to_yaml)
  end
end
