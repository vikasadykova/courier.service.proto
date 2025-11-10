openapi: 3.0.3
info:
  title: "Сервис управления курьерами"
  version: "1.0.0"
  description: "API для регистрации, обновления статуса и получения информации о курьерах"

tags:
  - name: "courier"
    description: "Методы для работы с курьерами"

paths:

  /couriers:
    get:
      summary: "Получить список курьеров"
      description: "Возвращает список курьеров с фильтрацией, сортировкой и пагинацией"
      tags:
        - "courier"
      parameters:
        - name: sort
          in: query
          schema:
            type: string
          example: "updated_at"
        - name: _order
          in: query
          schema:
            type: string
          example: "desc"
        - name: _page_size
          in: query
          schema:
            type: integer
          example: 20
        - name: status
          in: query
          schema:
            type: array
            items:
              type: string
          example: "available"
      responses:
        200:
          description: "Список курьеров"
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/Courier"
        400:
          $ref: "#/components/responses/ErrorResponse400"
        401:
          $ref: "#/components/responses/UnauthorizedError"
        403:
          $ref: "#/components/responses/ForbiddenError"
        500:
          $ref: "#/components/responses/ErrorResponse500"

    post:
      summary: "Зарегистрировать нового курьера"
      description: "Создаёт запись о новом курьере"
      tags:
        - "courier"
      requestBody:
        $ref: "#/components/requestBodies/RegisterCourierRequest"
      responses:
        201:
          $ref: "#/components/responses/CourierCreatedResponse"
        400:
          $ref: "#/components/responses/ErrorResponse400"
        401:
          $ref: "#/components/responses/UnauthorizedError"
        403:
          $ref: "#/components/responses/ForbiddenError"
        500:
          $ref: "#/components/responses/ErrorResponse500"

  /couriers/{courier_id}:
    get:
      summary: "Получить информацию о курьере"
      description: "Возвращает текущий статус и местоположение курьера"
      tags:
        - "courier"
      parameters:
        - name: courier_id
          in: path
          required: true
          schema:
            type: string
      responses:
        200:
          $ref: "#/components/responses/CourierStatusResponse"
        400:
          $ref: "#/components/responses/ErrorResponse400"
        401:
          $ref: "#/components/responses/UnauthorizedError"
        403:
          $ref: "#/components/responses/ForbiddenError"
        404:
          $ref: "#/components/responses/NotFoundError"
        500:
          $ref: "#/components/responses/ErrorResponse500"

    put:
      summary: "Обновить статус и локацию курьера"
      description: "Обновляет местоположение и занятость курьера"
      tags:
        - "courier"
      parameters:
        - name: courier_id
          in: path
          required: true
          schema:
            type: string
      requestBody:
        $ref: "#/components/requestBodies/UpdateCourierRequest"
      responses:
        200:
          $ref: "#/components/responses/CourierUpdatedResponse"
        400:
          $ref: "#/components/responses/ErrorResponse400"
        401:
          $ref: "#/components/responses/UnauthorizedError"
        403:
          $ref: "#/components/responses/ForbiddenError"
        404:
          $ref: "#/components/responses/NotFoundError"
        500:
          $ref: "#/components/responses/ErrorResponse500"

    delete:
      summary: "Удалить курьера"
      description: "Удаляет запись о курьере"
      tags:
        - "courier"
      parameters:
        - name: courier_id
          in: path
          required: true
          schema:
            type: string
      responses:
        204:
          $ref: "#/components/responses/NoContentResponse"
        400:
          $ref: "#/components/responses/ErrorResponse400"
        401:
          $ref: "#/components/responses/UnauthorizedError"
        403:
          $ref: "#/components/responses/ForbiddenError"
        404:
          $ref: "#/components/responses/NotFoundError"
        500:
          $ref: "#/components/responses/ErrorResponse500"

components:

  schemas:

    Courier:
      type: object
      properties:
        courier_id:
          type: string
        name:
          type: string
        status:
          type: string
          enum: [AVAILABLE, BUSY]
        latitude:
          type: number
          format: float
        longitude:
          type: number
          format: float
        updated_at:
          type: string
          format: date-time

    Error:
      type: object
      properties:
        errorId:
          type: string
        errorCode:
          type: string
        errorMessage:
          type: string
        errorDetails:
          type: object
          additionalProperties: true
      required:
        - errorId
        - errorMessage
        - errorCode

  requestBodies:

    RegisterCourierRequest:
      description: "Данные для регистрации нового курьера"
      required: true
      content:
        application/json:
          schema:
            type: object
            properties:
              name:
                type: string
              latitude:
                type: number
                format: float
              longitude:
                type: number
                format: float
            required:
              - name
              - latitude
              - longitude

    UpdateCourierRequest:
      description: "Данные для обновления статуса курьера"
      required: true
      content:
        application/json:
          schema:
            type: object
            properties:
              status:
                type: string
                enum: [AVAILABLE, BUSY]
              latitude:
                type: number
                format: float
              longitude:
                type: number
                format: float

  responses:

    CourierCreatedResponse:
      description: "Курьер успешно зарегистрирован"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Courier"

    CourierStatusResponse:
      description: "Информация о курьере"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Courier"

    CourierUpdatedResponse:
      description: "Курьер успешно обновлён"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Courier"

    NoContentResponse:
      description: "Курьер успешно удалён"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"

    UnauthorizedError:
      description: "Ошибка аутентификации"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"

    ForbiddenError:
      description: "Ошибка авторизации"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"

    NotFoundError:
      description: "Курьер не найден"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"

    ErrorResponse400:
      description: "Ошибка валидации запроса"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"

    ErrorResponse500:
      description: "Внутренняя ошибка сервера"
      headers:
        x-tracking-id:
          $ref: "#/components/parameters/XTrackingId"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Error"

  parameters:

    XTrackingId:
      name: x-tracking-id
      in: header
      description: "Уникальный идентификатор запроса, генерируемый сервером"
      required: true
      schema:
        type: string
