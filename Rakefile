# frozen_string_literal: true

require 'rake/clean'
require 'asciidoctor'
require 'asciidoctor-pdf'
require 'fileutils'

# Configuration
BASE_DIR = 'docs'
SOURCE_DIR = 'docs'
BUILD_DIR = 'build'
IMAGES_DIR = 'images'
HTML_THEME_DIR = 'html-theme'
PDF_THEME_DIR = 'pdf-theme/themes'
PDF_FONTS_DIR = 'pdf-theme/fonts'

# Common attributes for all documents.
# Attributes ending in '@' are soft-set, so a document header may override them
# (the Emerald Edict selects its own PDF theme, for example).
COMMON_ATTRIBUTES = {
  # Neither document uses an admonition or an icon macro, so the font-based
  # icons only bought a Font Awesome request from a CDN on every page view.
  'icons!' => '',
  'media' => 'screen',
  'compress' => '',
  'language' => 'EN',
  'imagesdir' => IMAGES_DIR,
  # The HTML theme is linked rather than embedded, so both documents share one
  # cached stylesheet. copy_theme puts it in the build directory next to the
  # webfonts and border tiles it references, so Asciidoctor must not copy it.
  'stylesdir' => '.',
  'stylesheet' => 'el.css',
  'linkcss' => '',
  'copycss!' => '',
  'pdf-themesdir' => File.expand_path(PDF_THEME_DIR),
  'pdf-fontsdir' => "#{File.expand_path(PDF_FONTS_DIR)},GEM_FONTS_DIR",
  'pdf-theme' => 'el@'
}.freeze

# Clean tasks
CLEAN.include("#{BUILD_DIR}/**/*.{html,pdf}")
CLOBBER.include(BUILD_DIR, "#{BASE_DIR}/**/build")

# Convert a single document to both HTML and PDF
def convert_document(source_file, output_dir)
  absolute_output = File.expand_path(output_dir)

  %w[html5 pdf].each do |backend|
    Asciidoctor.convert_file(
      source_file,
      safe: :unsafe,
      backend: backend,
      base_dir: BASE_DIR,
      to_dir: absolute_output,
      mkdirs: true,
      attributes: COMMON_ATTRIBUTES
    )
  end
end

# Copy files from source to destination and clean up source
def copy_and_cleanup(source_dir, dest_dir)
  FileUtils.mkdir_p(dest_dir)
  FileUtils.cp_r("#{source_dir}/.", dest_dir)
  FileUtils.rm_rf(source_dir)
end

# Task to copy the images referenced by the rendered HTML
desc 'Copy the images to the build directory'
task :copy_images do
  source = "#{SOURCE_DIR}/#{IMAGES_DIR}"
  target = "#{BUILD_DIR}/#{IMAGES_DIR}"

  if Dir.exist?(source)
    puts "Copying #{source}..."
    FileUtils.mkdir_p(target)
    FileUtils.cp_r("#{source}/.", target)
  else
    puts "Warning: #{source} not found"
  end
end

# Task to copy the HTML theme (stylesheet, webfonts, border tiles)
desc 'Copy the HTML theme to the build directory'
task :copy_theme do
  if Dir.exist?(HTML_THEME_DIR)
    puts "Copying #{HTML_THEME_DIR}..."
    FileUtils.mkdir_p(BUILD_DIR)
    FileUtils.cp_r("#{HTML_THEME_DIR}/.", BUILD_DIR)
  else
    puts "Warning: #{HTML_THEME_DIR} not found"
  end
end

# Task to render all rules documents
desc 'Render all rules documents to PDF and HTML'
task :render_rules_documents do
  build_dir = "#{SOURCE_DIR}/build"

  Dir.glob("#{SOURCE_DIR}/*.adoc").each do |document_file|
    puts "Converting #{document_file}..."
    convert_document(document_file, build_dir)
  end

  copy_and_cleanup(build_dir, BUILD_DIR)
end

# Main build task
desc 'Build all rules documents'
task :build => [:render_rules_documents, :copy_images, :copy_theme] do
  puts "\nBuild completed successfully!"
  puts "Output directory: #{BUILD_DIR}"
end

# Default task
task default: :build
