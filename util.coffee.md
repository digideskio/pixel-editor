Util
====

    require "extensions"
    require "./shims"

    global.Bindable = require "bindable"
    global.Matrix = require "matrix"
    global.Model = require "core"
    global.Point = require "point"
    global.Observable = require "observable"
    global.Size = require "size"

    Matrix.Point = Point

Helpers
-------

    componentToHex = (c) ->
      hex = c.toString(16)

      if hex.length is 1
        "0" + hex
      else
        hex

    isObject = (object) ->
      Object::toString.call(object) is "[object Object]"

Point Extensions
----------------

    Point.prototype.scale = (scalar) ->
      if isObject(scalar)
        Point(@x * scalar.width, @y * scalar.height)
      else
        Point(@x * scalar, @y * scalar)

    Point.prototype.floor = ->
      Point @x.floor(), @y.floor()

Extra utilities that may be broken out into separate libraries.

    module.exports =
      endDeltoid: (start, end) ->
        if end.x < start.x
          x = 0
        else
          x = 1

        if end.y < start.y
          y = 0
        else
          y = 1

        end.add(Point(x, y))

Call an iterator for each integer point on a line between two integer points.

      line: (p0, p1, iterator) ->
        {x:x0, y:y0} = p0
        {x:x1, y:y1} = p1

        dx = (x1 - x0).abs()
        dy = (y1 - y0).abs()
        sx = (x1 - x0).sign()
        sy = (y1 - y0).sign()
        err = dx - dy

        while !(x0 is x1 and y0 is y1)
          e2 = 2 * err

          if e2 > -dy
            err -= dy
            x0 += sx

          if e2 < dx
            err += dx
            y0 += sy

          iterator
            x: x0
            y: y0

      rect: (start, end, iterator) ->
        [start.y..end.y].forEach (y) ->
          [start.x..end.x].forEach (x) ->
            iterator
              x: x
              y: y

      rectOutline: (start, end, iterator) ->
        [start.y..end.y].forEach (y) ->
          if y is start.y or y is end.y
            [start.x..end.x].forEach (x) ->
              iterator
                x: x
                y: y
          else
            iterator
              x: start.x
              y: y

            iterator
              x: end.x
              y: y

gross code courtesy of http://en.wikipedia.org/wiki/Midpoint_circle_algorithm

      circle: (start, endPoint, iterator) ->
        center = Point.interpolate(start, endPoint, 0.5).floor()
        {x:x0, y:y0} = center
        {x:x1, y:y1} = endPoint

        radius = (endPoint.subtract(start).magnitude() / 2) | 0

        f = 1 - radius
        ddFx = 1
        ddFy = -2 * radius

        x = 0
        y = radius

        iterator x0, y0 + radius
        iterator x0, y0 - radius
        iterator x0 + radius, y0
        iterator x0 - radius, y0

        while x < y
          if f > 0
            y--
            ddFy += 2
            f += ddFy

          x++
          ddFx += 2
          f += ddFx

          iterator x0 + x, y0 + y
          iterator x0 - x, y0 + y
          iterator x0 + x, y0 - y
          iterator x0 - x, y0 - y
          iterator x0 + y, y0 + x
          iterator x0 - y, y0 + x
          iterator x0 + y, y0 - x
          iterator x0 - y, y0 - x

      rgb2Hex: (r, g, b) ->
        return "#" + [r, g, b].map(componentToHex).join("")
