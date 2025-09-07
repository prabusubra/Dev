package com.example.sqlgenerator;

import com.example.sqlgenerator.annotations.ColumnInfo;
import jakarta.persistence.Entity;
import jakarta.persistence.Table;

import java.lang.reflect.Field;
import java.util.*;
import java.util.stream.Collectors;

/**
 * Production-grade Query Builder for JPA Entities
 */
public class EntityQueryBuilder<T> {

    private final Class<T> rootEntity;
    private final String rootTable;
    private final String rootAlias;
    private final List<String> selectFields = new ArrayList<>();
    private final List<String> joins = new ArrayList<>();
    private final List<String> whereConditions = new ArrayList<>();
    private String orderBy;
    private Integer limit;
    private Integer offset;

    public EntityQueryBuilder(Class<T> entityClass, String alias) {
        validateEntity(entityClass);
        this.rootEntity = entityClass;
        this.rootTable = resolveTableName(entityClass);
        this.rootAlias = alias;
        extractSelectFields(entityClass, alias);
    }

    private void validateEntity(Class<?> entityClass) {
        if (!entityClass.isAnnotationPresent(Entity.class)) {
            throw new IllegalArgumentException("Class " + entityClass.getSimpleName() + " is not annotated with @Entity.");
        }
    }

    private String resolveTableName(Class<?> entityClass) {
        if (entityClass.isAnnotationPresent(Table.class)) {
            Table table = entityClass.getAnnotation(Table.class);
            return !table.name().isEmpty() ? table.name() : entityClass.getSimpleName();
        }
        return entityClass.getSimpleName();
    }

    private void extractSelectFields(Class<?> entityClass, String alias) {
        for (Field field : entityClass.getDeclaredFields()) {
            ColumnInfo colInfo = field.getAnnotation(ColumnInfo.class);
            String column = (colInfo != null) ? colInfo.column() : field.getName();
            String jsonName = (colInfo != null) ? colInfo.jsonName() : field.getName();
            selectFields.add(alias + "." + column + " AS " + jsonName);
        }
    }

    // --- WHERE ---
    public EntityQueryBuilder<T> where(String condition) {
        this.whereConditions.add(condition);
        return this;
    }

    // --- ORDER BY ---
    public EntityQueryBuilder<T> orderBy(String column, boolean ascending) {
        this.orderBy = column + (ascending ? " ASC" : " DESC");
        return this;
    }

    // --- LIMIT / OFFSET ---
    public EntityQueryBuilder<T> limit(int limit) {
        this.limit = limit;
        return this;
    }

    public EntityQueryBuilder<T> offset(int offset) {
        this.offset = offset;
        return this;
    }

    // --- JOINs ---
    public <J> EntityQueryBuilder<T> join(Class<J> joinEntity, String alias, String condition, String type) {
        validateEntity(joinEntity);
        String joinTable = resolveTableName(joinEntity);
        joins.add(type + " JOIN " + joinTable + " " + alias + " ON " + condition);
        extractSelectFields(joinEntity, alias);
        return this;
    }

    public <J> EntityQueryBuilder<T> innerJoin(Class<J> joinEntity, String alias, String condition) {
        return join(joinEntity, alias, condition, "INNER");
    }

    public <J> EntityQueryBuilder<T> leftJoin(Class<J> joinEntity, String alias, String condition) {
        return join(joinEntity, alias, condition, "LEFT");
    }

    public <J> EntityQueryBuilder<T> rightJoin(Class<J> joinEntity, String alias, String condition) {
        return join(joinEntity, alias, condition, "RIGHT");
    }

    // --- Many-to-Many ---
    public <J> EntityQueryBuilder<T> manyToManyJoin(
            Class<J> targetEntity,
            String targetAlias,
            String joinTable,
            String joinAlias,
            String leftCondition,
            String rightCondition) {

        validateEntity(targetEntity);

        // First join middle table
        joins.add("INNER JOIN " + joinTable + " " + joinAlias + " ON " + leftCondition);

        // Then join target entity
        String targetTable = resolveTableName(targetEntity);
        joins.add("INNER JOIN " + targetTable + " " + targetAlias + " ON " + rightCondition);

        extractSelectFields(targetEntity, targetAlias);

        return this;
    }

    // --- BUILD QUERY ---
    public String build() {
        StringBuilder query = new StringBuilder();
        query.append("SELECT ")
             .append(String.join(", ", selectFields))
             .append(" FROM ").append(rootTable).append(" ").append(rootAlias);

        if (!joins.isEmpty()) {
            query.append(" ").append(String.join(" ", joins));
        }
        if (!whereConditions.isEmpty()) {
            query.append(" WHERE ").append(String.join(" AND ", whereConditions));
        }
        if (orderBy != null) {
            query.append(" ORDER BY ").append(orderBy);
        }
        if (limit != null) {
            query.append(" LIMIT ").append(limit);
        }
        if (offset != null) {
            query.append(" OFFSET ").append(offset);
        }
        query.append(";");
        return query.toString();
    }
}
