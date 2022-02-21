require './constants'

desc 'create MBTiles'
task :tiles do
  sh <<-EOS
gdalwarp -dstnodata None #{TIF_FN} a.tif; \
docker run -ti --rm -v #{HERE_DIR}:#{MOUNT_DIR} unvt/rgbify \
rasterio rgbify -b -10000 -i 0.1 --max-z 16 --min-z 8 \
--format webp --verbose \
#{MOUNT_DIR}/a.tif #{MOUNT_DIR}/#{MBTILES_FN}; \
rm a.tif
  EOS
end

desc 'create style.json'
task :style do
  sh <<-EOS
charites build --provider mapbox style.yml docs/style.json
  EOS
end

desc 'host the site locally'
task :host do
  sh <<-EOS
budo -d docs
  EOS
end

desc 'deploy tiles'
task :deploy do
  require 'sqlite3'
  DB = SQLite3::Database.new(MBTILES_FN)
  DB.execute("SELECT zoom_level, tile_column, tile_row, tile_data FROM tiles") {|zoom_level, tile_column, tile_row, tile_data|
    y = 2 ** zoom_level - tile_row - 1
    path = "docs/zxy/#{zoom_level}/#{tile_column}/#{y}.webp"
    dir = File.dirname(path)
    sh "mkdir -p #{dir}" unless File.directory?(dir)
    next if tile_data.size <= 54
    File.open(path, 'wb') {|w|
      print "#{path} #{tile_data.size}\n"
      w.write(tile_data)
    }
  }
end

